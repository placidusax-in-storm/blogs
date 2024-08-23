# Lerna bootstrap

Link local packages together and install remaining package dependencies

When run, this command will:

1. npm install all external dependencies of each package.
2. Symlink together all Lerna packages that are dependencies of each other.
3. npm run prepublish in all bootstrapped packages (unless --ignore-prepublish is passed).
4. npm run prepare in all bootstrapped packages.
