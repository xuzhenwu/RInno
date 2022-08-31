# changed features
1. Install Inno, need to install in the website 

https://jrsoftware.org/isdl.php#stable

2 . add new version support 

get_R.R fix 1-3 to 1-9
code_section.R fix  1-3 to 1-9

sa suggested by 
https://github.com/ficonsulting/RInno/pull/156/files


# RInno <img src="man/figures/RInno.png" align="right" height=140/>

[![AppVeyor Build
Status](https://ci.appveyor.com/api/projects/status/github/ficonsulting/RInno?branch=master&svg=true)](https://ci.appveyor.com/project/ficonsulting/RInno)
[![codecov](https://codecov.io/github/ficonsulting/RInno/branch/master/graphs/badge.svg)](https://codecov.io/github/ficonsulting/RInno)
[![CRAN\_Status\_Badge](https://www.r-pkg.org/badges/version/RInno)](https://cran.r-project.org/package=RInno)
[![Downloads](https://cranlogs.r-pkg.org/badges/RInno)](https://cran.rstudio.com/package=RInno)
[![Downloads](https://cranlogs.r-pkg.org/badges/grand-total/RInno)](https://cran.rstudio.com/package=RInno)
[![Project Status: Active - The project has reached a stable, usable
state and is being actively
developed.](http://www.repostatus.org/badges/latest/active.svg)](http://www.repostatus.org/#active)

RInno makes it easy to install local shiny apps by providing an
interface between R, [Inno Setup](http://www.jrsoftware.org/isinfo.php),
an installer for Windows programs (sorry Mac and Linux users), and
[Electron](https://electronjs.org/), a modern desktop framework used by
companies like Github, Slack, Microsoft, Facebook and Docker. RInno is
designed to be simple to use (two lines of code at a minimum), yet
comprehensive.

If a user does not have R installed, the RInno installer can be
configured to ask them to install R along with a shiny app, `include_R =
TRUE`. And similar to Dr. Lee Pang’s
[DesktopDeployR](https://github.com/wleepang/DesktopDeployR) project,
RInno provides a framework for managing software dependencies and error
logging features. However, RInno also supports GitHub package
dependencies, continuous installation (auto-update on start up), and it
is easier to manage with `create_app`, the main RInno function.
DesktopDeployR requires many manual adjustments and a deep understanding
of the entire framework to use, but RInno can be learned incrementally
and changes automatically flow down stream. You don’t need to remember
the 100+ places impacted by changing `app_dir`. RInno only requires a
high-level understanding of what you’d like to accomplish.

## Getting Started

    # Get remotes package
    install.packages("remotes"); require(remotes)
    
    # Use install_github to get RInno
    install_github("ficonsulting/RInno")
    
    # Require Package
    require(RInno)
    
    # Use RInno to get Inno Setup
    install_inno()

## Minimal example

Once you have developed a shiny app, you can build an installer with
`create_app` followed by `compile_iss`.

    # Example app included with RInno package
    example_app(app_dir = "app")
    
    # Build an installer
    create_app(app_name = "Your appname", app_dir = "app")
    compile_iss()

`create_app` creates an installation framework in your app’s directory,
`app_dir`. The main components are a file called “app\_name.iss” and the
“nativefier-app” directory. You can perform minor customizations before
you call `compile_iss`. For example, you can replace the default/setup
icon at [Flaticon.com](http://www.flaticon.com/), or you can customize
the pre-/post- install messages, *infobefore.txt* and *infoafter.txt*.
Just remember, the default values (i.e. `create_app(info_after =
"infobefore.txt")`) for those files have not changed. The Inno Setup
Script (ISS), *app\_name.iss*, will look for *default.ico* and try to
use it until you update the script or call `create_app` with the new
icon’s file name (i.e. `create_app(app_icon = "new.ico")`). Likewise,
the Electron app will need to be recompiled to capture any manual
changes to files in `app_dir`.

Electron is now used to render the shiny app’s UI. All other
`user_browser` options will be deprecated in future releases.

## ui.R Requirements

In order to replace Electron’s logo with your app’s icon, add something
like this to your ui.R file:

    fluidPage(
      tags$head(
        tags$link(
          rel = "icon", 
          type = "image/x-icon", 
          href = "http://localhost:1984/default.ico")
      )
    )

## server.R Requirements

In order to close the app when your user’s session completes:

1.  Add `session` to your `server` function
2.  Call `stopApp()` when the session ends

<!-- end list -->

    function(input, output, session) {
    
      if (!interactive()) {
        session$onSessionEnded(function() {
          stopApp()
          q("no")
        })
      }
    }

If you forget to do this, users will complain that their icons are
broken and rightly blame you for it (an R session will be running in the
background hosting the app, but they will need to press ctrl + alt +
delete and use their task manager to close it). **Not cool**.

## Package Dependency Management

Provide a named character vector of packages to `create_app`, and RInno
will download them and install them with your shiny app. RInno downloads
windows binaries from CRAN for the listed packages and their
dependencies with `tools::package_dependencies(packages = pkgs,
recursive = TRUE)`.

    create_app(
      app_name = "myapp", 
      app_dir = "app",
      pkgs = c("shiny", "jsonlite", "httr")
    )

For `remotes`, Github source files are compiled into windows binaries.
Bitbucket will be supported in a future release.

## Custom Installations

If you would like to create a custom installer from within R, you can
slowly build up to it with `create_app`, like this:

    create_app(
      app_name    = "My AppName", 
      app_dir     = "My/app/path",
      dir_out     = "wizard",
      pkgs        = c("jsonlite", "shiny", "magrittr", "xkcd"),  # CRAN-like repo packages
      remotes     = c("talgalili/installr", "daattali/shinyjs"), # GitHub packages
      include_R   = TRUE,     # Download R and install it with your app, if necessary
      R_version   = "2.2.1",  # Old versions of R
      privilege   = "high",   # Admin only installation
      default_dir = "pf")     # Install app in to Program Files

`create_app` passes its arguments to most of the other support functions
in RInno. You can (and probably should) specify most things there and
they will get passed on. Alternatively, you can provide instructions
directly to those support functions like
    this:

    # Copy installation scripts (JavaScript, icons, infobefore.txt, package_manager.R, launch_app.R)
    copy_installation(app_dir = "my/app/path")
    
    # If your users need R installed:
    get_R(app_dir = "my/app/path", R_version = "2.2.1")
    
    # Create batch file
    create_bat(app_name = "My AppName", app_dir = "my/app/path")
    
    # Create app config file
    create_config(app_name = "My AppName", R_version = "2.2.1", app_dir = "my/app/path",
      pkgs = c("jsonlite", "shiny", "magrittr", "dplyr", "caret", "xkcd"))
    
    # Build the iss script
    start_iss(app_name = "My AppName") %>%
    
      # C-like directives
      directives_section(R_version   = "2.2.1", 
                 include_R   = TRUE,
                 app_version = "0.1.2",
                 publisher   = "Your Company", 
                 main_url    = "yourcompany.com") %>%
    
      # Setup Section
      setup_section(output_dir  = "wizard", 
            app_version = "0.1.2",
            default_dir = "pf", 
            privilege   = "high",
            inst_readme = "pre-install instructions.txt", 
            setup_icon  = "myicon.ico",
            pub_url     = "mycompany.com", 
            sup_url     = "mycompany.github.com/issues",
            upd_url     = "mycompany.github.com") %>%
    
      # Languages Section
      languages_section() %>%
    
      # Tasks Section
      tasks_section(desktop_icon = FALSE) %>%
    
      # Files Section
      files_section(app_dir = "my/app/path", file_list = "path/to/extra/files") %>%
    
      # Icons Section
      icons_section(app_desc       = "This is my local shiny app",
            app_icon       = "notdefault.ico",
            prog_menu_icon = FALSE,
            desktop_icon   = FALSE) %>%
    
      # Execution & Pascal code to check registry during installation
      # If the user has R, don't give them an extra copy
      # If the user needs R, give it to them
      run_section() %>%
      code_section() %>%
    
      # Write the Inno Setup script
      writeLines(file.path("my/app/path", "My AppName.iss"))
    
      # Check your files, then
      compile_iss()

Feel free to read the Inno Setup
[documentation](http://www.jrsoftware.org/ishelp/) and RInno’s
documentation to get a sense for what is possible. Also, please suggest
useful features or build them yourself\! We have a very positive culture
at FI Consulting, and we would love to get your feedback.

Please note that this project has a [Contributor Code of
Conduct](https://github.com/ficonsulting/RInno/blob/master/CONDUCT.md).
By participating in this project you agree to abide by its terms.

## License

The RInno package is licensed under the GPLv3. See LICENSE for details.
