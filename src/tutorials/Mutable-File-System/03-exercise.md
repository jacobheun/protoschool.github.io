Let's add some files to IPFS using MFS. Since `files` is an array representing the files currently available to us in the browser, we'll need to loop through the array and use `files.write()` to add each file we find there to IPFS, one at a time. (Only have a single file uploaded? No problem!)

Put your files in the root directory (`/`) and be sure to add the name of each file to the path when you add it (**hint:** the file object in the browser stores the filename as `file.name`). Be sure to set up your options so that a new file is created when one isn't found at the given path.
