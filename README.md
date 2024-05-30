# Grafana Export

Grafana exporter is a modified csv exporter that outputs csv data in a format that works well with csv import into Grafana dashboards.

# Parameters

## Plugin config

Required fields:

- `output-path`: The full file path to write the exported csv file.

Optional fields:

- `headers`: A list of headers to extract from the inputs to write as columns in the csv file. An empty list of headers will write all fields provided as inputs to the `pluginParams`. The default list of headers is empty therefore all input data will be written to the csv file.

## Inputs

The inputs should be in the standard format provided by the IF project.

## Outputs

This plugin will write externally to disk as csv file and pass the inputs directly as output.

## Implementation

To run the plugin, you must first create an instance of `GrafanaExport` and call its `execute()` function with the desired inputs.

## Usage

To run the `grafana-export`, an instance of `PluginInterface` must be created. Then, the plugin's `execute()` method can be called, passing required arguments to it.

This is how you could run the model in Typescript:

```typescript
async function runPlugin() {
    const newPlugin = GrafanaExport();
    const usage = await newPlugin.execute([
        {
            timestamp: '2023-07-06T00:00',
            duration: 1,
            'operational-carbon': 0.02,
            'carbon-embodied': 5,
            energy: 3.5,
            carbon: 5.02,
        },
    ]);

    console.log(usage);
}

runPlugin();
```

## Testing model integration

### Using local links

For using locally developed model in `IF Framework` please follow these steps: 

1. On the root level of a locally developed model run `npm link`, which will create global package. It uses `package.json` file's `name` field as a package name. Additionally name can be checked by running `npm ls -g --depth=0 --link=true`.
2. Use the linked model in impl by specifying `name`, `method`, `path` in initialize models section. 

```yaml
name: grafana-export-demo
description: example exporting output to a csv file
tags:
initialize:
  outputs:
    - yaml
  plugins:
    grafana-exporter:
      method: GrafanaExport
      path: 'grafana-export'
tree:
  children:
    child:
      pipeline:
        - grafana-exporter
      config:
        grafana-exporter:
          output-path: C:/dev/grafana-export.csv
          headers:
            - timestamp
            - duration
            - carbon
            - energy
      inputs:
        - timestamp: 2023-07-06T00:00
          duration: 1
          carbon-operational: 0.02
          carbon-embodied: 5
          energy: 3.5
          carbon: 5.02
        - timestamp: 2023-07-06T00:10
          duration: 1
          operational-carbon: 0.03
          carbon-embodied: 4
          energy: 2.9
          carbon: 4.03
```

### Using directly from Github

You can simply push your model to the public Github repository and pass the path to it in your impl.
For example, for a model saved in `github.com/perkss/grafana-export` you can do the following:

npm install your model: 

```
npm install -g https://github.com/perkss/grafana-export
```

Then, in your `impl`, provide the path in the model instantiation. You also need to specify which class the model instantiates. In this case you are using the `PluginInterface`, so you can specify `OutputModel`. 

```yaml
name: grafana-export-demo
description: example exporting output to a csv file
tags:
initialize:
  outputs:
    - yaml
  plugins:
    grafana-exporter:
      method: GrafanaExport
      path: https://github.com/perkss/grafana-export
tree:
  children:
    child:
      pipeline:
        - grafana-exporter
      config:
        grafana-exporter:
          output-path: C:/dev/grafana-export.csv
          headers:
            - timestamp
            - duration
            - carbon
            - energy
      inputs:
        - timestamp: 2023-07-06T00:00
          duration: 1
          carbon-operational: 0.02
          carbon-embodied: 5
          energy: 3.5
          carbon: 5.02
        - timestamp: 2023-07-06T00:10
          duration: 1
          operational-carbon: 0.03
          carbon-embodied: 4
          energy: 2.9
          carbon: 4.03
```

Now, when you run the `manifest` using the IF CLI, it will load the model automatically. Run using:

```sh
ie --manifest <path-to-your-impl> --output <path-to-save-output>
```
