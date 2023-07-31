# idf-examples-launchpad-ci-action
This Github Action builds merged binaries of ESP examples and uploads them to the Github pages for ESP Launchpad.

# Usage

## Step 1: Setup github pages in your repository
You will have to setup Github pages in your repository. The binaries will be uploaded to the `gh-pages` branch of your repository.

## Step 2: Configuration file
It is important to have `.idf_build_apps.toml` configuration file in the root directory of your repository. This file is used to configure the build process. 
For further information, please refer to the [documentation](https://docs.espressif.com/projects/idf-build-apps/en/latest/config_file.html).

Example configuration file:
```toml
paths = "examples" # Paths to search for buildable projects ["examples", "components"]
target = "all" # esp32, esp32s2, esp32c3, esp32s3, all...
recursive = true # Search for buildable projects recursively on the paths

# Configuration file for the build process 
config = "sdkconfig.defaults"
```
**Do not overwrite** `collect-app-info` **and** `build-dir` **options!**

## Inputs
### `github_token`
**Required.** The Github token to use for uploading the binaries.

### `idf_version`
The version of the ESP-IDF to use for building the binaries.
**Default:** `"latest"`

### `parallel_count`
The number of parallel builds.
**Default:** `1`

### `parallel_index`
The index of the parallel build.
**Default:** `1`

## Step 3: You will have to run the action on a container with ESP-IDF installed. 
### Example usage

Basic workflow:

```yaml
jobs:

  build-and-upload-binaries:
    strategy:
      matrix:
        idf_ver: ["latest"]
    runs-on: ubuntu-latest
    container: espressif/idf:${{ matrix.idf_ver }}
    steps:
      - name: Checkout repo
        uses: actions/checkout@v3
        with:
          submodules: 'recursive'

      - name: Action for Building and Uploading Binaries
        uses: XDanielPaul/idf-examples-launchpad-ci-action@v1.0
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          idf_version: ${{ matrix.idf_ver }}
```

Workflow with parallel builds:

```yaml
jobs:
  build-and-upload-binaries:
    strategy:
      matrix:
        include:
          - idf_ver: "latest"
            parallel_count: 2
            parallel_index: 1
          - idf_ver: "release-v4.4"
            parallel_count: 2
            parallel_index: 2
    runs-on: ubuntu-latest
    container: espressif/idf:${{ matrix.idf_ver }}
    steps:
      - name: Checkout repo
        uses: actions/checkout@v3
        with:
          submodules: 'recursive'

      - name: Action for Building and Uploading Binaries
        uses: XDanielPaul/idf-examples-launchpad-ci-action@v1.0
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          idf_version: ${{ matrix.idf_ver }}
          parallel_count: ${{ matrix.parallel_count }}
          parallel_index: ${{ matrix.parallel_index }}
```

## Step 4: Switch github pages to the `gh-pages` branch
After the action has finished, you will have to switch the Github pages to the `gh-pages` branch. You can do this in the settings of your repository.
