# loopback-4-ngx-build-sdk

This is an example setup for creating angular 8 sdk from loopback 4 app using openapi spec and openapi-generator-cli.

# Test the setup

1. Clone this repository.

2. Install the dependencies and start the server.

```sh
# install the dependencies:
npm run install;

# start the loopback 4 server
npm start;
```

3. Run the script to create the typescript angular sdk

```sh
# build sdk
npm run sdk:build;

```

You should now get `./sdk` folder containing typescript apis and models ready
to integrate in a ngx app.

# How can I do the same on my Loopback 4 app?

Those are the steps I did to setup the process for building angular sdk.

## Install openapi generator cli

```sh
# install the latest version of "openapi-generator-cli"
npm install @openapitools/openapi-generator-cli -save-dev
```

## Add build-ngx-sdk.sh to loopback root

Copy the shell script from this repository `build-ngx-sdk.sh` to the root directory of your loopback 4 app.

This is the script:

```sh

############# download the openapi.json ################
curl http://\[::1\]:3000/explorer/openapi.json \
  --output openapi.json

############## create the sdk ##################
./node_modules/.bin/openapi-generator generate \
  -i openapi.json \
  -g typescript-angular \
  -o sdk \
  --additional-properties=\"ngVersion=8.3.1\"

```

The script does two things:

1. it downloads the OpenApi specs
2. it creates the typescript angular sdk

Modify the parameters in the script as needed.

## Add script to package.json

In package.json of the loopback root folder add those two scripts:

```jsonc
 "scripts": {

   // ...

    "sdk:build": "bash build-ngx-sdk.sh",
   // ...

  }
```

## Modify .gitignore

Add `openapi.json` to .gitignore.

You should be ready...

## Run the script to create the typescript angular sdk

```sh

# start server
npm start;

# build sdk
npm run sdk:build;

```

# Remarks

If you are [mounting loopback 3 in loopback 4](https://loopback.io/doc/en/lb4/migration-mounting-lb3app.html) and looking for a replacement for `mean-expert-official/loopback-sdk-builder`, you may run into these issues using the above approach:

## The models generated do only contain the own properties but not the related models.

The easiest workaround I found: Add the related models as properies in loopack 3 model definitions, e.g.:

```jsonc
// looback3/common/models/foo.json
{
  "name": "Foo",

  // ...

  "relations": {
    "bar": {
      "type": "hasMany",
      "model": "Bar",
      "primaryKey": "id",
      "foreignKey": "foo_id"
    }
  },

  // ...

  "properties": {
    "id": {
      "type": "number",
      "id": true,
      "generated": true
    },
    "name": {
      "type": "string"
    },
    // Add bar as property to make it appear in openapi spec and sdk !!
    // use RelatedModel or [RelatedModel] as type, depending on cardinality
    "bar": {
      "type": ["Bar"]
    }
  }

  // ...
}
```

This made the related models appear in api explorer, in openapi.json and also in the resulting sdk.
