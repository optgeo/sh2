REGION = 'europe'
AREA = 'albania'

desc 'download source geospatial data to the place'
task :download do
  u = "https://download.geofabrik.de/#{REGION}/{#{AREA}-latest.osm.pbf}"
  sh "curl -C - #{u} --output './src/#1'"
end

desc 'build tiles from source data'
task :tiles do
  sh "osmium extract --overwrite --polygon=extent/extent.geojson --output=src/extract.osm.pbf src/#{AREA}-latest.osm.pbf"
  sh "osmium export --config conf/osmium-export-config.json --index-type=sparse_file_array --output-format=geojsonseq --output=- src/extract.osm.pbf | node conf/filter.js | tippecanoe --no-feature-limit --no-tile-size-limit --force --simplification=2 --maximum-zoom=15 --base-zoom=15 --hilbert --output=tiles.mbtiles"
  sh "tile-join --force --no-tile-compression --output-to-directory=docs/zxy --no-tile-size-limit tiles.mbtiles"
end

desc 'host the site'
task :host do
  sh "budo -d docs"
end

desc 'build style.json from HOCON descriptions'
task :style do
  require 'json'
   sh "parse-hocon hocon/style.conf > docs/style.json"
  center = JSON.parse(File.read('docs/zxy/metadata.json'))['center'].split(',')
    .map{|v| v.to_f }.slice(0, 2)
  style = JSON.parse(File.read('docs/style.json'))
  style['center'] = center
  File.write('docs/style.json', JSON.pretty_generate(style))
  sh "gl-style-validate docs/style.json"
end

# This Rakefile is a version of https://github.com/unvt/naru

