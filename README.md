# CSEE8300_Project
Automatically Building Executable Files for RaspberryPi with pyinstaller


Using this repository as a guide to build executable files is pretty simple. Navigate to .github/workflows/main.yml and add this file to your repository (it needs to have the same path as well)

Once you have copied the file and its contents simply edit them as needed for your project.

```yaml
on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]
  workflow_dispatch:
```

  This defines what events you want the regression to run on. Currently it runs on push requests, pull requests and on demand (workflow_dispatch).
  
``` yaml 
env:
  file_to_build: algo_VTCN.py #file you want to build
  pi_image: raspios_lite_arm64:latest #image you want to use need 64 bit image for pytorch
  path_to_file: CSEE8300_Project/exampleCode #folder that $file_to_build is in
  built_file_name: algo_VTCN #name of the built file
```

These are the environment variables for the file. I chose to use environment variables here so it's obvious what some of the options you may want to change are.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout repo #checkout code so most up-to-date version is accessible to the runner
      uses: actions/checkout@main
      with: 
        lfs: true #lfs is github large file storage system
    - name: checkout lfs object
      run: git lfs checkout
    - name: Build test
      uses: pguyot/arm-runner-action@main #this action emulates an ARM CPU and a desired operating system $pi_image
      id: Build-Python-exe
      with:
        image_additional_mb: 3584
        copy_repository_path: /home/pi
        copy_artifact_path: ${{env.path_to_file}}/dist/${{env.built_file_name}}
        base_image: ${{env.pi_image}}
        commands: |
            sudo apt-get update -y;
            pwd;
            apt-get install -y python3 python3-venv python3-dev python3-pip; 
            cd ${{env.path_to_file}};
            pip3 install torch;
            pip3 install --upgrade pip;
            pip3 install numpy --upgrade;
            pip3 install -U scikit-learn;
            pip3 install -U matplotlib;
            pip3 install pyinstaller;
            pip3 install nvidia-ml-py3
            pyinstaller --clean --onefile ${{env.file_to_build}};
    - uses: actions/upload-artifact@main
      with:
        name: exe_file
        path: ${{env.built_file_name}}
        if-no-files-found: error
        retention-days: 3
```

This is jobs section and it defines what you want the runner to do when the events above happen. If you have edited the environment variables at the top of the file as suggested the only thing you need to edit here are the packages that need to be installed to build the program. Please note that pyinstaller is required to be installed for the program to be built. This job is specifically building a raspberry pi image with python3 installed as well as all of the packages installed. The upload-artifact section is grabbing the specific file we wanted built and uploading it to github.

Finding the packages compatible with raspberry pi is likely what will cause the most problems with building the program. For instance the default Pi OS is 32 bit and pytorch will not run on a 32 bit operating system and the error codes given does not make it immediately obvious what the error is so some trial and error is required. Some tips: stack overflow is your friend adding the flags to some of these installs was done at the suggestion of posts I found on stack overflow and without these flags they would not install. Also, ensure the package is compatible with the desired operating system.

```yaml
     - name: Compress the release image
       run: |
         sudo xz -T 0 -v ${{ steps.Build-Python-exe.outputs.image }}
     - name: Upload the image artifact
       uses: actions/upload-artifact@v3
       with:
         name: build_image
         path: ${{ steps.Build-Python-exe.outputs.image }}.xz
         if-no-files-found: error
         retention-days: 10
```

Include these lines if you would also like the build image to be uploaded to github once the job has been completed. This can be very useful if you have many devices you wish to run the program or that all require python and specific packages added to them.


```yaml

  upload:
    runs-on: raspi
    needs: build
    steps: 
      - name: Checkout repo
        uses: actions/checkout@main
      - uses: actions/download-artifact@main
        with:
          name: exe_file
      - run: cp ${{env.built_file_name}} /home/pi/
      - run: cd /home/pi
      - run: chmod +x ${{env.built_file_name}}
```

You can add this job if your device is connected to the internet after you install a self-hosted runner to the device. To add a self hosted runner your repository should be made private then to go settings>actions>runners and then 'add self hosted runner' and follow the steps provided on the page. Be sure to add a tag that is meaningful to the machine such as I did 'raspi'. This tells github which machine to run the job on if you have more than one self-hosted runners.
