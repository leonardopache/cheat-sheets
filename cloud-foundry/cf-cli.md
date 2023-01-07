
```sh
# Login to cloud Foundry Instance
cf login --sso
``` 
```sh
# Select Space target from the Organization
cf target [-o ORG] [-s SPACE]
```
```sh
# Go into the app's project directory
cd APP_NAME
```
```sh
# Deploy the application
cf push APP_NAME
```
```sh
# Restarts the application 
cf restart APP_NAME
```
```sh
# Restages the application
cf restage APP_NAME
```
```sh
# Display recent logs and tasks
cf logs APP_NAME --recent
```
```sh
# Scales an app process with a count number
cf scale APP_NAME --process pn -i [COUNT]
```
```sh
# Display the buildpacks available
cf buildpacks
```