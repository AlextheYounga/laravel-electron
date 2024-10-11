# Laravel Electron for Desktop Applications
## Automatically Download a PHP binary and Package Electron with PHP and Laravel

<p align="center">
<img src="https://gblobscdn.gitbook.com/spaces%2F-LBKK1y7h_XWAtuRJG9X%2Favatar.png?alt=media" alt="ElectronJS" width="200"/>
<img src="https://laravel.com/img/logomark.min.svg" alt="Laravel Logo" width="170"/> 
</p>

**Wouldn't it be cool if you could add a folder called `desktop` to your Laravel application, and BOOM, it's now a desktop application? Well you can.**

> Specs:\
> OS: Mac and Linux\
> PHP: versions 8.1, 8.2, 8.3\
> Node: confirmed on Node 18.17

### Quick Start
1. Create a new Laravel application
```shell
laravel new example_laravel_11_app
# Fill out the prompt fields
```

2. Clone this repository into your Laravel application
```shell
cd ./example_laravel_11_app

# Clone this repo into your laravel application under "desktop" folder
git clone https://github.com/AlextheYounga/laravel-electron ./desktop
```

Move into the desktop folder and install packages
```shell
cd ./desktop
nvm install # Ensure correct version of node
npm install -g yarn # Ensure yarn is installed
yarn # Install electron packages
```

3. Check the `desktop-config.json` file
```json
{
	"appUrl": "http://localhost:8124",
	"autoStartPHP": true
}
```

This file will also be packaged with the Electron app under `extraResources`, so anything you put in here stays with the build.

By default, we set the app url to `localhost` at `8124`, but you can also change this to an external server if you'd like. 

If you do set this to anything other than `localhost`, my build script is smart enough to skip packaging Laravel and PHP into the Electron app, and just creates a bare-bones Electron wrapper that points to your server URL. 

4. Build
   
Run `yarn build`

```shell
LaravelElectron: Packaging Laravel app and PHP binaries for localhost build.

Running on macOS
Running on arm64 architecture
Input a PHP version (Supported versions: 8.1 | 8.2 | 8.3):
```

### How Does This Work?
The build script will automatically determine your architecture and prompt you for a PHP version *(8.1, 8.2, 8.3)*. We then pull the appropriate PHP binary from [NativePHP](https://github.com/NativePHP/php-bin/) and unpack it. 

When running `yarn build`, we tell Electron to take a `git` clone of our outer Laravel application and copy it into `desktop/laravel`. We then run some important scripts, like `yarn build` and `composer install`, just to make sure everything is up to date. We're then going to copy the `vendor` folder and `public/build` folder into `desktop/laravel/vendor` and `desktop/laravel/public/build` respectively. From here, we have everything we need to build our app.

In `package.json` we use the following resources to build our Electron app. 
```json
"extraResources": [
	{
		"from": "./desktop-config.json",
		"to": "build/desktop-config.json"
	},
	{
		"from": "./php/php",
		"to": "build/php/php"
	},
	{
		"from": "./laravel",
		"to": "build/laravel",
		"filter": [
			"**/*",
			".git",
			"!desktop",
			"!tests"
		]
	}
],
```

These will eventually be stored under Electron's resources folder, which we can access from our main process using:
```js
const electronResources = process.resourcesPath;
// Returns path to electron's resource folder.

const laravel = path.join(electronResources, 'build/laravel'); // Packaged laravel app folder
const php = path.join(electronResources, 'build/php/php'); // Path to PHP binary
```

What makes this eventually all work is the following inside of `main.js`. We just start a PHP server using our PHP binary whenever the app starts up.

```js
// Start php server
exec(`${php} -S localhost:8124 -t ${laravel}/public`);
```


### Development Testing
Note that the typical `yarn start` electron script will not work as expected unless you change one minor thing:
- Change the `autoStartPHP` value to `false` in `desktop-config.json`.
- Then run `php artisan serve --port 8124`
- Then you can run `yarn start` and it will point to your dev server as expected.

The reason here is that `yarn start` is not going to package your laravel app into Electron's assets, and it will be trying to auto start a PHP binary that doesn't exist within Electron's dev environment, to point to a Laravel application which also doesn't exist. By turning autoStartPHP to false, we are telling Electron not to worry about trying to start your own PHP version, just point to my dev server. 

### Icons
I included some icons I made with the `electron-icon-maker` package, included in `package.json`. This took me a very long time to figure out how to configure properly, so I'm keeping these in here to help guide you. I also remember spending at least a day or two just trying to size the images properly. 

### Known Issues

- Shutting down PHP (lol). I never thought far enough into this to figure out how best to kill PHP when the app closes. So for now, the PHP server just keeps running indefinitely in the background. PHP isn't a typically heavy program. I saw that it was using no CPU and about 7MB of RAM at idle. This is something that will surely come to me in my sleep eventually. 

