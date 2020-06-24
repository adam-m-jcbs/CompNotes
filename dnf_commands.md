A list of `dnf` (package manager used by Fedora Linux) commands I find useful.

#### List enabled repos, all
`dnf repolist [all]`

#### Add repo
`dnf config-manger --add-repo="<repo url>"`

#### Search for packages
`dnf search <package name>`

#### Find the provider of something
`dnf provides <system file>`

#### Install rpm
`dnf install file.rpm`

#### Directory for custom repos on Fedora systems
`/etc/yum.repos.d/`

#### Clean cache
`dnf clean all`
#### A less destructive option if you just want to make sure metadata is updated
`dnf update --refresh`
