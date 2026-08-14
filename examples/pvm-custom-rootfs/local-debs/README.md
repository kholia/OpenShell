# Local Debian packages

Place internal `.deb` files in this directory before creating the sandbox. Git
ignores everything here except this README and `.gitignore`.

The Dockerfile passes all local packages to `apt-get install` together, so apt
can resolve dependencies from the configured Ubuntu repositories. The package
archives are removed from the final rootfs after installation. Their installed
files and the copied archives can remain recoverable from OCI image layers.
Keep the resulting image in an access-controlled local daemon or private
registry.

https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
