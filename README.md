# HMOS-SDK-Setup
HarmonyOS SDK setup for GitHub Actions


## Creating the Github Actions workflow.
1) After repo is created and all the code is added. 
   click on the Actions in github UI and select "set up a work flow yourself"

   ![image](https://user-images.githubusercontent.com/11833821/123408783-08f0ac80-d5cb-11eb-9ff0-4c4d8e5faefe.png)

3) Add the below code into the file
```xml   
name: Build
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
jobs:
  Hmos_Build:
    runs-on: windows-latest
    steps:
    - uses: actions/checkout@v2
    - name: Set up JDK 11
      uses: actions/setup-java@v2
      with:
        java-version: '11'
        distribution: 'adopt'
    - name: Grant execute permission for gradlew
      run: chmod +x gradlew    
    - name: set environment variables
      uses: allenevans/set-env@v2.0.0
      with:
          OHOS_SDK_HOME: '${{ github.workspace }}\huawei'
    - name: Download HMOS SDK
      uses: actions/checkout@main
      with:
        repository: applibgroup/HarmonyOsSdk
        path: huawei
    - name: Cache SonarCloud packages
      uses: actions/cache@v1
      with:
        path: ~/.sonar/cache
        key: ${{ runner.os }}-sonar
        restore-keys: ${{ runner.os }}-sonar
    - name: Cache Gradle packages
      uses: actions/cache@v1
      with:
        path: ~/.gradle/caches
        key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle') }}
        restore-keys: ${{ runner.os }}-gradle
    - name: Hmos Build and analyze code
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}  # Needed to get PR information, if any
        SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
      run: ./gradlew assembleDebug sonarqube --info
    - name: Upload Artifact
      uses: actions/upload-artifact@v2
      with: 
        name: assets-for-download
        path: build\outputs\hap\debug\phone
  checkstyle:
    name: checkstyle
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: dbelyaev/action-checkstyle@master
        with:
          github_token: ${{ secrets.github_token }}
          reporter: github-check
          level: warning
```

## Adding the changes to build.gradle
1) Edit build.gradle in the root directory and add the below lines of code

```xml 
apply plugin: 'org.sonarqube'


sonarqube {
  properties {
    property "sonar.projectKey", "applibgroup_CircularProgressView"
    property "sonar.organization", "applibgroup"
    property "sonar.host.url", "https://sonarcloud.io"
    property "sonar.sources", "entry,circularprogressview"
    property "sonar.java.binaries", "entry/build,circularprogressview/build"
  }
}
```

### After Every commit the buid will be triggered. Click on the Actions to see the build status. 
### The Code analysis result will be available at the following location for all Applibgroup projects.
https://sonarcloud.io/organizations/applibgroup/projects
