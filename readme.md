# Compile a single executable from your Node app with Node.js 20 and ESBuild
[Compile a single executable from your Node app with Node.js 20 and ESBuild](https://dev.to/chad_r_stewart/compile-a-single-executable-from-your-node-app-with-nodejs-20-and-esbuild-210j)

## Introduction
Node.js 20 was released very recently. Along with several other features, you can now compile your Node.js project into a single executable that can be run in environments without Node.js installed. It’s important to note that this is still experimental and may not be suitable for use in production.

Node.js has instructions on how to set up these single executables: https://nodejs.org/api/single-executable-applications.html

Unfortunately, when compiling the executable, you will not compile dependencies into your executable. To solve this problem, we will leverage a JavaScript bundler to bundle our dependencies into one file before compiling it into our single executable.

## Putting Together our Project
First, we need a project that we will build into our executable.

We’ll first define our server.

Voir fichiers présents dans ce projet.

## Creating the Single Binary File
Begin by installing all the dependencies we’ll need by running this command:
```
yarn
```

Once npm install is completed, we run the command:
```
yarn build
```

This will create our server-out.js which will be our bundled file we will make into an executable.

Note: If you rather, you can follow the instructions from the Node.js guide starting from step 3 as the following steps will be exactly the same, located here: https://nodejs.org/api/single-executable-applications.html

Generate the blob to be injected:
```
node --experimental-worker sea-config.json
```

Create a copy of the node executable and name it according to your needs:
```
cp $(command -v node) server
```

Note: If you are on a Linux Distro (such as Ubuntu), you can skip the next steps and move straight to running the binary.

Remove the signature of the binary:

On macOS:
```
codesign --remove-signature server
```

On Windows (optional):
signtool can be used from the installed Windows SDK. If this step is skipped, ignore any signature-related warning from postject.
```
signtool remove /s server
```

Inject the blob into the copied binary by running postject with the following options:

* server - The name of the copy of the node executable created in step 2.
* NODE_SEA_BLOB - The name of the resource / note / section in the binary where the contents of the blob will be stored. sea-prep.blob - The name of the blob created in step 1.
* --sentinel-fuse NODE_SEA_FUSE_fce680ab2cc467b6e072b8b5df1996b2 - The fuse used by the Node.js project to detect if a file has been injected.
* --macho-segment-name NODE_SEA (only needed on macOS) - The name of the segment in the binary where the contents of the blob will be stored.

To summarize, here is the required command for each platform:

On systems other than macOS:
```
npx postject server NODE_SEA_BLOB sea-prep.blob \
    --sentinel-fuse NODE_SEA_FUSE_fce680ab2cc467b6e072b8b5df1996b2
```

On macOS:
```
npx postject server NODE_SEA_BLOB sea-prep.blob \
    --sentinel-fuse NODE_SEA_FUSE_fce680ab2cc467b6e072b8b5df1996b2 \
    --macho-segment-name NODE_SEA 
```

Sign the binary:

On macOS:
```
codesign --sign - server 
```

On Windows (optional):
A certificate needs to be present for this to work. However, the unsigned binary would still be runnable.
```
signtool sign /fd SHA256 server 
```

Run the binary:
```
./server server-out.js
```

You should now have a running Node server similar if you just ran node server-out.js

If you wanted to see a completed example, go here: https://github.com/chadstewart/ts-node-executable-article-example

Si vous ne souhaitez pas copier l'exécutable node, vous pouvez directement lancer le serveur avec la commande suivante :
```
/usr/bin/node server-out.js
```
