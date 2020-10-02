# Reproduction steps

- `npm i -g @vue/cli`
- `npm ci`
- `vue invoke vue-cli-plugin-sample`

The last command will fail because @vue/cli uses yaml-front-matter 3, whose `loadFront` accepts either file contents, or a string to a file path. @vue/cli only uses this with file contents, BUT the bug happens if the file contents just happens to be a file path.

In this project, I have a sample-plugin module which has a generator for vue-cli to use. In its template, a simple .eslintignore file (_eslintignore because its a dot file). This file just has `node_modules`, which would be normal for eslint, but because the path `node_modules` exists, yaml-front-matter tries to read the contents of a folder... which doesn't go well:

```
 ERROR  Error: EISDIR: illegal operation on a directory, read
Error: EISDIR: illegal operation on a directory, read
    at Object.readSync (fs.js:524:3)
    at tryReadSync (fs.js:349:20)
    at Object.readFileSync (fs.js:386:19)
    at Object.jsYaml.loadFront (/home/jsanders/.nvm/versions/node/v12.16.3/lib/node_modules/@vue/cli/node_modules/yaml-front-matter/lib/js-yaml-front.js:29:19)
    at renderFile (/home/jsanders/.nvm/versions/node/v12.16.3/lib/node_modules/@vue/cli/lib/GeneratorAPI.js:495:23)
    at /home/jsanders/.nvm/versions/node/v12.16.3/lib/node_modules/@vue/cli/lib/GeneratorAPI.js:300:27
    at processTicksAndRejections (internal/process/task_queues.js:97:5)
    at async Generator.resolveFiles (/home/jsanders/.nvm/versions/node/v12.16.3/lib/node_modules/@vue/cli/lib/Generator.js:268:7)
    at async Generator.generate (/home/jsanders/.nvm/versions/node/v12.16.3/lib/node_modules/@vue/cli/lib/Generator.js:175:5)
    at async runGenerator (/home/jsanders/.nvm/versions/node/v12.16.3/lib/node_modules/@vue/cli/lib/invoke.js:111:3)
```

# Proposed fix

Make @vue/cli use version 4 of yaml-front-matter. Breaking changes: https://github.com/dworthen/js-yaml-front-matter/blob/master/CHANGELOG.md#breaking-changes

Should be safe to simply update because vue-cli only uses `loadFront` function to process file contents. Version 4 only allows file contents to be processed, where as version 3 allowed file paths (hence the breaking change). 
