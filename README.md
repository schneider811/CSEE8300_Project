# CSEE8300_Project
Automatically Building Executable Files for RaspberryPi with pyinstaller


Using this repository as a guide to build executable files is pretty simple. Navigate to .github/workflows/main.yml and add this file to your repository (it needs to have the same path as well)

Once you have copied the file and its contents simply edit them as needed for your project.

'''
on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]
  workflow_dispatch:
'''

  This defines what events you want the regression to run on. Currently it runs on push requests, pull requests and on demand (workflow_dispatch).
  
'''  
env:
  file_to_build: algo_VTCN.py #file you want to build
  pi_image: raspios_lite_arm64:latest #image you want to use need 64 bit image for pytorch
  path_to_file: CSEE8300_Project/exampleCode #folder that $file_to_build is in
  built_file_name: algo_VTCN #name of the built file
'''
