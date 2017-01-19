# Introducing Slap: Simple Library for Automated Publishing

We had a client recently who wanted to automate publication of some map services in ArcGIS (for) Server via a scheduled task; this is a very common need, and we've developed a tool to help manage publication.

For the purposes of this example, assume we have the following setup:

* An ArcGIS for Server instance, `http://myhost.mydomain.com`
* An admin user on the instance, with username `siteadmin`, password `arcgis`
* A directory `C:\maps` on `myhost`, containing three map documents that we want to publish: `map1.mxd`, `map2.mxd`, and `map3.mxd`

## Setting up the Environment

Typically you'll run slap from your ArcGIS server; depending on how your system is set up, there are a couple of task that may need to be done first:

* The local Python installation must be in the system `PATH` so that it can be run from a command prompt
* The default Python package manager (pip) must be available

If you're not sure how to set these things up, see [Setting up a new Python environment with ArcGIS](link).  Once your Python environment is set up properly, installing slap is easy; just open a comand prompt and do:

```shell
pip install slap
```

## Configuration 
Slap uses a configuration file to determine where and how to publish map services; this configuration file is in standard [json](https://www.copterlabs.com/json-what-it-is-how-it-works-how-to-use-it/) format, and can be generated using slap itself.  To create the bare-bones configuration file that `slap` will use to publish services, open your command prompt and do:

```shell
cd c:\maps
slap init --name myhost.mydomain.com
```

You should see a new file in the directory, named `config.json`, with a list of inputs to publish:

```javascript
{                                                   
    "agsUrl": "https://myhost.myname.com:6443/arcgis/admin", 
    "mapServices": {                                
        "services": [                               
            {                                       
                "input": "C:\\maps\\map1.mxd"       
            },                                      
            {                                       
                "input": "C:\\maps\\map2.mxd"       
            },                                      
            {                                       
                "input": "C:\\maps\\map3.mxd"       
            }                                       
        ]                                           
    }                                               
}                                                   
```

### Editing the Config File
You can open the `config.json` file with any text editor, and make changes to the entries; for example, if you want to publish `map1.mxd` to a subfolder called `myMaps` and change the service name to `myMapService`, change the config file to:

```javascript
{                                                   
    "agsUrl": "https://myhost.myname.com:6443/arcgis/admin", 
    "mapServices": {                                
        "services": [                               
            {                                       
                "input": "C:\\maps\\map1.mxd",
                "folder": "myMaps",
                "serviceName": "myMapService"
            },                                      
            {                                       
                "input": "C:\\maps\\map2.mxd"       
            },                                      
            {                                       
                "input": "C:\\maps\\map3.mxd"       
            }                                       
        ]                                           
    }                                               
}                                                   
```

For more configuration examples, see the slap [documentation](https://github.com/gisinc/slap#config-files).

## Publishing Services

Once the config file is set up, you can publish the new services by doing:

```
slap publish --username siteadmin --password argis
```


