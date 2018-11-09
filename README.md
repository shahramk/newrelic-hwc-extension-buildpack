
# New Relic HWC Extension Buildpack

## Introduction
New Relic HWC extension buildpack uses the multi-buildpack approach and works with Cloud Foundry's standard HWC buildpack to stage and run your Dotnet Framework applications. It installs and configures New Relic Dotnet Framework agent, and it then passes the control to the standard Cloud Foundry HWC buildpack to install other application dependencies in the droplet.

 


## How New Relic HWC Extension Buildpack Works
The buildpack obtains New Relic Dotnet Framework agent by using one of the following ways as fits in your environment:

1. If an environment variable named **"NEW_RELIC_DOWNLOAD_URL"** is defined in the application's environment (manifest file, cf set-env, or set in AppsMgr), its value is used as the location to download the agent. This is used for environments where you have your own repository for downloading dependencies (i.e. Artifactory). If a second environment variable called **"NEW_RELIC_DOWNLOAD_SHA256"** is set in the application environment, it is expected to hold the downloaded agent's binary file SHA256 checksum. In this case the SHA256 checksum of the downloaded agent binary file is compared with this value, and in case of mismatch, the buildpack reports an error.

1. You could create a cached version of the buildpack using the **"--cached"** switch with New Relic agent embedded in the buildpack. This is mainly used in disconnected (isolated) environments where PCF does not have access to the outside world, and the buildpack cannot download the agent from New Relic's download site.

1. Use a specific version of New Relic agent by specifying the version in the buildack's manifest file at the time of packaging.

1. Set the version of the agent to **"0.0.0.0"**, **"latest"**, or **"current"** in buildpack's manifest file to download the latest version of the agent.

In the 3rd and 4th cases (and in the 1st case when SHA256 checksum is provided), the buildpack checks the SHA256 checksum of the downloaded agent to validate the download.

The order of precedence for which method to use to obtain New Relic agent is from the top to bottom. If **"NEW_RELIC_DOWNLOAD_URL"** is specified, it precedes the other options. If this environment variable is not specified, the cached buildpack takes precedence. Otherwise, the **"version"** property of the agent in the buildpack's manifest is used, and one of the other two options is used to download the agent, depending on the value of **"version"** property of the agent dependency (explicit version or "latest").

We always encourage you to use the latest version of New Relic agents unless there is a reason that keeps you from upgrading to newer releases of the agent.


## How The Extension Buildpack Operates
The buildpack looks for several environment variables and files to determine how to bind the application to the agent.


### Bind the Application to New Relic Agent
The license key for New Relic account must be specified in one of the following ways:
* NEW_RELIC_LICENSE_KEY env var
* License key from User-Provided-Service
* License key from Service Broker Tile in Marketplace
* License key from newrelic.config

**Note:** environment variables override all other options.


### Application Name in New Relic UI
The application name for New Relic is determined in the following order:
* NEW_RELIC_APP_NAME env var
* App name from User-Provided-Service
* App name from newrelic.config
* App name from PCF


### New Relic Agent Configuration File
New Relic configuration file (**"newrelic.config"**) would allow you to set a number of agent properties, and change the behavior of the agent as you wish. Refer to [.NET agent configuration][a] for more information on configuring the agent. You could make a copy of this file into the application's root directory, and change any of agent's settings. The buildpack looks for the agent's config file in the following order:
* App root folder
* Buildpack folder
* Agent folder


### New Relic User-Provided-Services
If the application binds to a User-Provided-Service with the word **"newrelic"** as part of its name, the buildpack sets the credentials from this service in the application environment by setting environment variable for known New Relic properties. The known properties currently are:
* NEW_RELIC_LICNESE_KEY
* NEW_RELIC_APP_NAME
* NEW_RELIC_DISTRIBUTED_TRACING_ENABLED


### Use of Proxy
If you're using a proxy server in your environment, you need to make a copy of **"newrelic.config"** file of the agent in the application directory, and specify the [proxy information][b] as a child of the **<service>** element. 
**Example**
```bash
<service licenseKey="0123456789abcdef0123456789abcdef01234567"> 
  <proxy name="my.proxy.address" port="proxy.port" />
</service>
```


## Deploying the application
At a minimum you need to have **CF CLI version 6.38** in order to push your application using multi buildpacks. You could either use the [**buildpacks** stanza][c] in the application manifest file, or you can use multiple **"-b"** options in the **"cf push"** command line to specify multiple buildpacks. Refer to the ***Example** section below to see more details. **Please note that Cloud Foundry's main HWC buildpack must be the last buildpack in the chain.**


## Building the Buildpack
To build this extension buildpack, run the following commands from the buildpack's directory:

1- Source the **.envrc** file in the buildpack directory.
```bash
source .envrc
```
To simplify the process in the future, install [direnv][d] which will automatically source **.envrc** when you change directories.

2- Install **buildpack-packager**
```bash
./scripts/install_tools.sh
```

3- Build the buildpack
```bash
buildpack-packager build [ --cached ] [ -any-stack ] [ -stack cflinuxfs2 | cflinuxfs3 ]
```

## Use The Extension Buildpack in Cloud Foundry
To use the buildpack in Cloud Foundry
Upload the buildpack to your Cloud Foundry and optionally specify it by name

```bash
cf create-buildpack [NEWRELIC_HWC_EXTENSION_BUILDPACK] [BUILDPACK_ZIP_FILE_PATH] 99
cf push my_app -b NEWRELIC_HWC_EXTENSION_BUILDPACK   -b HWC_BUILDPACK
```

## Example: Push a Sample Dotnet Framework Application to PCF
The following example shows the steps to configure the app manifest to bind a Dotnet Framework application New Relic Dotnet Framework agent using New Relic HWC extension buildpack and Cloud Foundry's standard HWC buildpack, and pushes the app to PCF. Note that the example uses environment variable to specify the license key for the New Relic account. You could explore this page for alternative ways to bind an application to the agent.

1- Create a sample dotnet framework application (or pick one of your dotnet framework apps)

2- Test the app locally and make sure it runs with no errors

3- Create a manifest.yml file for the application<br/>
<pre>
	---
	applications:
	- name: myapp
	  memory: 256M  # adjust as needed
	  buildpacks:
	    - nr_hwc_extension_buildpack
	    - hwc_buildpack
	  env:
	    NEW_RELIC_LICENSE_KEY: 0123456789abcdef0123456789abcdef01234567

</pre>

4- Push the application to PCF<br/>
```
cf push
```







### Reporting Issues
Open an issue at [support.newrelic.com](https://support.newrelic.com/)



[a]: https://docs.newrelic.com/docs/agents/net-agent/configuration/net-agent-configuration
[b]: https://docs.newrelic.com/docs/agents/net-agent/configuration/net-agent-configuration#proxy
[c]: https://docs.cloudfoundry.org/devguide/deploy-apps/manifest.html#buildpack
[d]: https://direnv.net/
