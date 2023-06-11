# Custom Web HMI Demo - PLCNext Engineer Project

## What is This?

### This Repo

This is a basic example of custom webpages to show how you could make an
advanced and fully custom web HMI beyond what's possible with the default HMI
pages tools in PLCNext Engineer (eHMI).

This project uses mainly:

* [Vue.js](https://vuejs.org/), a modern JavaScript frontend framework to make
the dynamic pages.

* [JavaScript Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)
to interact with the controller's REST API to exchange information about the PLC
variables.

This aims to be a basic minimal example.

### Custom Web HMI Demo Project

This is part of an example project trying to show how you can make webpages with
modern web development tools, upload them to a PLCNext controller (physical or
simulated during development and testing), and have the controller's webserver
serve them.

It is much more involved than a traditional "what you see is what you get" HMI
editor (like the default HMI Pages in PLCNext Engineer, called "eHMI"), but the
possibilities to make much more advanced and customised projects are endless.
You are not limited to a particular framework or set of tools either.

This project is not made or endorsed by Phoenix Contact or PLCNext themslves,
this is just code being shared as a guide/example/tutorial by a user for other
users.

## How to use This

* At the top of the base repo folder in GitHub, use the options in "<> Code" to
download these source files: either clone the repository or download it as .zip.

* You will need to have [Node.js](https://nodejs.org/en) installed to be able to
use npm (Node Package Manager): Open a console in the repo's folder in your
machine and do `npm install` to have npm automatically download dependencies.

* To load the webpage files to the controller, first you need to build the
project to have Vue and Vite generate the needed files:
  * `npm run devbuild` will build the project in debug mode, allowing more
    debug options (like using Vue devtools in the browser) but in a real project
    it should NOT be used for production.
  * `npm run build` will build the project in production mode, optimizing the
    files for deploying to production if this were a real project.

* The generated files will be put in `/dist` folder. You'll have to create a
`custweb` folder in `/opt/plcnext/projects/PCWE/Services/Ehmi` and copy the dist
files inside. It's important to respect this path (or change project settings
accordingly, else Vue won't work properly!).

* Open a browser and navigate to
[https://\<controller address\>/custweb](https://\<controller address\>/custweb).
For example, [https://127.0.0.1:5050/custweb](https://127.0.0.1:5050/custweb)
for a simulated controller on the local computer.
