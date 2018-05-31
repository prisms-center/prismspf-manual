# Installing CMake and deal.II {#install_prereqs}

Before downloading and installing PRISMS-PF itself, one should install CMake and deal.II.

## Installing CMake
CMake can be downloaded at: https://cmake.org/download, which also has installation instructions. Be sure to add CMake to your path by entering:

`$ PATH="/path/to/cmake/Contents/bin":"$PATH" `

filling in the path to the installation of CMake (for example, on Mac OS, the default installation path is /Applications/CMake.app/Contents/bin). For convience, we recommend adding this line to your bash profile. Your bash profile can be opened via the following terminal command

`$ vi ~/.profile`


## Installing deal.II
deal.II can be downloaded at https://www.dealii.org/download.html, which also has instructions. The deal.II installation process depends on your operating system. PRISMS-PF has been tested with deal.II versions 8.4.2 through 9.0.0. We recommend using deal.II 9.0.0, if possible.

### Installing deal.II on Mac OS X
A binary package for the latest version of deal.II (and other versions) is available at [here](https://github.com/dealii/dealii/releases). The binary package includes all the libraries that you need (excluding CMake). To install deal.II open the .dmg file and drag ''deal.II.app'' into your Applications folder. Then, open ''deal.II.app'', which will open a Terminal window and set the necessary environment variables. You may need to modify your security settings so that your operating system will let you open the application (in System Preferences, select Security & Privacy, then under the general tab choose to allow apps downloaded from anywhere; don't click ''Open deal.II.app anyway'', we have found that doing so doesn't actually launch deal.II.app).

In some cases, deal.II.app will not open, possibly freezing at the ''Verifying...'' stage. If this happens, right click on it and select ''Show Package Contents''. Then go to ''/Contents/Resources/bin/'' and launch ''dealii-terminal''. This should launch the Terminal window that sets the environment variables.

We have found that for some older versions of Mac OS are incompatible with deal.II version 8.4.2 (due to the version of the Clang compiler that is installed by default). In those cases we recommend downloading [this version 8.3.0 package](https://github.com/dealii/dealii/releases/download/v8.3.0/dealii-8.3.0.nocxx14.dmg).
