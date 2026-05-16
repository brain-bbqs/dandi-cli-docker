# dandi-cli-docker
Containerized DANDI CLI wrapper for downloading or uploading a file with
environment variable flags.

## Build

```bash
docker build -t dandi-cli-env .
```

The Dockerfile installs a temporary compiler toolchain because some platforms
do not have prebuilt wheels for all DANDI dependencies, such as `numcodecs`.

## Download

DANDI's documented download command is just `dandi download <url>`. This
container keeps that shape and passes `DANDI_URL` directly to the CLI.

```bash
docker run --rm \
  -e DANDI_ACTION=download \
  -e DANDI_URL="DANDI:000023" \
  -e OUTPUT_DIR=/output \
  -v "$PWD/data:/output" \
  dandi-cli-env
```

`DANDI_URL` can be a DANDI identifier, API URL, web URL, or `dandi://` URL that
the DANDI CLI supports.

## Upload

Upload mode follows the EMBER/DANDI staging flow: download the draft Dandiset,
organize files from `INPUT_DIR`, validate the staged Dandiset, then upload to
the `ember-dandi` instance.

```bash
docker run --rm \
  -e DANDI_ACTION=upload \
  -e DATASET_ID=000123 \
  -e DANDI_API_KEY="$DANDI_API_KEY" \
  -e INPUT_DIR=/input \
  -v "$PWD/source_folder:/input:ro" \
  dandi-cli-env
```

The upload command run inside the container is equivalent to:

```bash
dandi download "https://dandi.emberarchive.org/dandiset/<DATASET_ID>/draft"
cd "<DATASET_ID>"
dandi organize "$INPUT_DIR" -f dry
dandi organize "$INPUT_DIR"
dandi validate .
dandi upload -i ember-dandi
```

## Environment Variables

| Variable | Default | Description |
| --- | --- | --- |
| `DANDI_ACTION` | unset | Required. Must be `download` or `upload`. |
| `DANDI_URL` | unset | URL or identifier passed to `dandi download`. Required for downloads. |
| `OUTPUT_DIR` | `/output` | Directory where downloads are saved. |
| `DATASET_ID` | unset | Dandiset identifier used to download the draft Dandiset before upload. `DANDISET_ID` is also accepted. |
| `DANDISET_URL` | `https://dandi.emberarchive.org/dandiset/<DATASET_ID>/draft` | Draft Dandiset URL used during upload staging. |
| `INPUT_DIR` | `/input` | Mounted source folder passed to `dandi organize` during upload. |
| `UPLOAD_WORK_DIR` | `/work` | Temporary staging directory for the downloaded draft Dandiset. |
| `DANDI_API_KEY` | unset | DANDI API token. Required for uploads. The entrypoint maps this to the instance-specific variable, such as `EMBER_DANDI_API_KEY` for `ember-dandi`. |
| `DANDI_INSTANCE` | `ember-dandi` | Instance passed to `dandi upload -i`. |
