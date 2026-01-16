## Editing a Dependency Name in a `.deb` Package

To change a dependency name inside a `.deb` package, you are **editing package metadata**, not the binary payload itself. Debian packages store dependency information in control metadata, which can be safely modified and rebuilt.

## Package structure overview

A `.deb` file is an `ar` archive containing three main parts:

-   **`control.tar.*`**  
    Package metadata (Package, Version, Depends, Maintainer, etc.)
    
-   **`data.tar.*`**  
    The actual files installed on the system
    
-   **`debian-binary`**  
    Format version indicator
    

Dependency declarations live in the **`control`** file inside `control.tar.*`, for example:
```
Depends: libc6 (>= 2.31), foo | bar
```
## Method: Edit the dependency directly (recommended)

### 1\. Extract the `.deb` package
```
dpkg-deb -R package.deb package-edit
```
This creates a directory structure similar to:
```
package-edit/
├── DEBIAN/
│   └── control
├── usr/
└── ...
```
### 2\. Edit the control file

Open the control file:
```
nano package-edit/DEBIAN/control
```
Modify the dependency name as needed:
```
- Depends: libfoo (>= 1.2) + Depends: libfoo2 (>= 1.2)
```
#### Important syntax rules

-   Commas (`,`) separate dependencies
    
-   Pipes (`|`) specify alternative dependencies
    
-   Version constraints must be enclosed in parentheses
    

Incorrect syntax will cause the package to fail installation.

### 3\. Rebuild the `.deb` package
```
dpkg-deb -b package-edit new-package.deb
```
### 4\. Fix ownership and permissions (optional but strongly recommended)

Debian packaging policy expects all files in a package to:

-   Be owned by `root:root`
    
-   Not be writable by group or others
    

Run:
```
chmod -R go-w package-edit
chown -R root:root package-edit
```
Then rebuild the package again:
```
dpkg-deb -b package-edit new-package.deb
```
Failing to do this may result in warnings, policy violations, or rejected packages in stricter environments.
