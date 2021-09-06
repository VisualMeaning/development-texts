# Getting tiles and data from Illustrator to the SMP

A basic guide on how to get tile and semantic labels exports from Adobe Illustrator processed and uploaded into the SMP. This is the upload process as of 2021-09-06.

This assumes you have AWS access via the AWS CLI, Github access, and the `tile-illustration`, `map-tools`, `transforms` and `sheet-to-triples` repos set up locally.

## Get Illustrator exports from S3

Illustrator exports labels .json and .pdfs with the tiles and overlays to a folder in our S3 bucket `opatlas-dev`.

Sync these files onto your local machine:

    aws s3 sync s3://opatlas-dev/helen/20210906133950/ poleco/

## Slicing PDFs into tiles

You need the `tile-illustration` repo cloned locally for this.

Tile slicing is currently done via unversioned (!!!) bash scripts that look like this:

<details>
  <summary>Click for code</summary>

```bash
#!/bin/sh
set -ex

L0="~0-2_tiles.pdf"
L3="~3-4_tiles.pdf"
L5="~5-6_tiles.pdf"

pdftocairo -singlefile -png -scale-to   256 poleco/"${L0}" poleco/~0
pdftocairo -singlefile -png -scale-to   512 poleco/"${L0}" poleco/~1
pdftocairo -singlefile -png -scale-to  1024 poleco/"${L0}" poleco/~2
pdftocairo -singlefile -png -scale-to  2048 poleco/"${L3}" poleco/~3
pdftocairo -singlefile -png -scale-to  4096 poleco/"${L3}" poleco/~4
pdftocairo -singlefile -png -scale-to  8192 poleco/"${L5}" poleco/~5

FMT=png OUTDIR=poleco-out ZOOMLEVEL=0 ../tile-illustration/split.sh poleco/~0.png
FMT=png OUTDIR=poleco-out ZOOMLEVEL=1 ../tile-illustration/split.sh poleco/~1.png
FMT=png OUTDIR=poleco-out ZOOMLEVEL=2 ../tile-illustration/split.sh poleco/~2.png
FMT=png OUTDIR=poleco-out ZOOMLEVEL=3 ../tile-illustration/split.sh poleco/~3.png
FMT=png OUTDIR=poleco-out ZOOMLEVEL=4 ../tile-illustration/split.sh poleco/~4.png
FMT=png OUTDIR=poleco-out ZOOMLEVEL=5 ../tile-illustration/split.sh poleco/~5.png

(cd poleco-out&&aws s3 sync . s3://opatlas-live/police-eco/20210906/tiles/)
```
</details>

The exact content will vary based on the map in question and the number of zoom levels that it has. The important line is at the bottom:

    (cd poleco-out&&aws s3 sync . s3://opatlas-live/police-eco/20210906/tiles/)

Once the tiles have all been sliced, this uploads the tiles back up to S3, this time to the `opatlas-live` bucket. Each version of the tiles is versioned in a different folder inside this bucket with the day's datestamp. 

### Edit the version datestamp in the bash script

**If we are creating new tiles, we need to create a new version folder for them inside S3**. This is done by editing the datestamp in the bash script to either today's date, or (if multiple runs on the same date) to

    (cd poleco-out&&aws s3 sync . s3://opatlas-live/police-eco/20210906v2/tiles/)

### Execute the bash script

Once this is done, the script can be executed.

    ./poleco.sh

### Additional tilesets

Some maps have additional tilesets for lenses/overlays that also need to be sliced and uploaded. The process is identical to that for a regular tileset, just with a different bash script that copies the output tiles to subfolders inside the version folder for the main tileset. 

<details>
  <summary>Click for code</summary>

```bash
#!/bin/sh
set -ex

LC="poleco"
LR="poleco-out-lens"

LENSES="present future"

for OV in ${LENSES}
do

L0="~${OV}_tiles.pdf"
OUT="${LR}/${OV}"

TRANSP="-transp"

pdftocairo -singlefile ${TRANSP} -png -scale-to   256 ${LC}/"${L0}" ${LC}/0
pdftocairo -singlefile ${TRANSP} -png -scale-to   512 ${LC}/"${L0}" ${LC}/1
pdftocairo -singlefile ${TRANSP} -png -scale-to  1024 ${LC}/"${L0}" ${LC}/2
pdftocairo -singlefile ${TRANSP} -png -scale-to  2048 ${LC}/"${L0}" ${LC}/3
pdftocairo -singlefile ${TRANSP} -png -scale-to  4096 ${LC}/"${L0}" ${LC}/4
pdftocairo -singlefile ${TRANSP} -png -scale-to  8192 ${LC}/"${L0}" ${LC}/5

FMT=png OUTDIR=${OUT} ZOOMLEVEL=0 ../tile-illustration/split.sh ${LC}/0.png
FMT=png OUTDIR=${OUT} ZOOMLEVEL=1 ../tile-illustration/split.sh ${LC}/1.png
FMT=png OUTDIR=${OUT} ZOOMLEVEL=2 ../tile-illustration/split.sh ${LC}/2.png
FMT=png OUTDIR=${OUT} ZOOMLEVEL=3 ../tile-illustration/split.sh ${LC}/3.png
FMT=png OUTDIR=${OUT} ZOOMLEVEL=4 ../tile-illustration/split.sh ${LC}/4.png
FMT=png OUTDIR=${OUT} ZOOMLEVEL=5 ../tile-illustration/split.sh ${LC}/5.png

done

(cd ${LR}&&aws s3 sync . "s3://opatlas-live/police-eco/20210906v2/overlays/")
```
</details>

**Remember to edit the version datestamp inside the script first.** Then run the script.

    ./poleco-lens.sh

## Label generation

Part of the export from Illustrator is a .json file containing the semantic labels. These need to be converted into an intermediary transform file that can be parsed as part of the data ingest in the next step.

This is handled by a script in the `map-tools` repository called `labels.py`. Run it:

    ./labels.py -s --name-layer='' ../pol_scratch/poleco/Landscape_draft_v2.5_-_for_discussion.json > ../transforms/labels/poleco_labels.py

## Data ingest

### Aggregation

The `sheet-to-triples` repository handles data aggregation, aggregating labels data, ontology data and any additional data from Excel workbooks etc. together into a single set of map terms.

**!Important!** If the map contains any additional tilesets that are displayed as part of a view, then the tileset location will need to be changed inside the file that defines the views in the `transforms` repository -- remember that we just created a new version of these tiles, so **we need to update the view definition to point to the new version**. Views are usually defined inside the `/extras` folder, for example in [poleco-defs.ttl](https://github.com/VisualMeaning/transforms/blob/master/extras/poleco_defs.ttl).

Once this has been done, we can run the data aggregation:

    TRANSFORMS_DIR=../transforms/labels/ python3 -m sheet_to_triples --model ~/backups/bare.json poleco_labels --add-graph ../transforms/extras/poleco_defs.ttl --add-graph ../transforms/extras/poleco_onto.ttl

This will create a .json file of all of the aggregated data called `new.json`.

### Upload

`new.json` is then loaded into the SMP via a script in `map-tools` called `restore.py`.

    python restore.py --backup ../sheet-to-triples/new.json --username alightwing --password $(cat ~/.credentials/vm) --base https://eom.ecosystem.guide --to http://visual-meaning.com/rdf/maps/police/ecosystem

## Update tileset in map configuration

Done via the Django admin panel at https://eom.ecosystem.guide/admin. Go to the Map config panel for the map being updated, and update the version datestamp in the `tiles_src` parameter to point to the new version that we sliced and uploaded earlier.
