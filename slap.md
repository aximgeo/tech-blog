# Introducing slap: Simple Library for Automated Publishing

We had a client recently who wanted to automate publication of some map services in ArcGIS (for) Server via a scheduled task.

## Setting up the Environment

Typically you'll run slap from an ArcGIS server instance; depending on how your system is set up, there are a couple of task that may need to be done first:

* The local Python installation must be in the system `PATH` so that it can be run from a command prompt
* The default Python package manager (pip) must be available

If you're not sure how to set these things up, see [Setting up a new Python environment with ArcGIS](link).  Once your Python environment is set up properly, installing slap is easy; just do:

```
pip install slap
```

## Configuration 
`slap` uses a configuration file to determine where and how to publish map services; this configuration file is in standard [json](https://www.copterlabs.com/json-what-it-is-how-it-works-how-to-use-it/) format, and can be generated using `slap` itself.

### Creating a Config File
To create the bare-bones configuration file that `slap` will use to publish services, just navigate to the directory where your map documents are stored, and do:

```
slap init
```

You should see a new file in the directory, named `config.json`, with a list of inputs to publish:

```
config example 1
```

### Editing the Config File
Open the `config.json` file with any text editor, and change the `agsUrl` value to match your ArcGIS server hostname; you can also change the URL to use HTTPS on port 6443 if you prefer:

```
"agsUrl": "https://myhost.mydomain.com:6443/arcgis/server"
```

You can add additional properties to the config file as needed; for example, you can publish service to subfolders on the server, and change the service names:

```
config example 2
```

For more configuration examples, see the slap [documentation](https://github.com/gisinc/slap#config-files).

## Publishing Services

Once the config file is set up, you can publish the new services by doing:

```
slap publish --username <publisher username> --password <publisher password>
```


