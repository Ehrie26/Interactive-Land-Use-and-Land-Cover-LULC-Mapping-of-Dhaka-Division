# Interactive-Land-Use-and-Land-Cover-LULC-Mapping-of-Dhaka-Division


ui.root.clear();

var leftMap = ui.Map();
var rightMap = ui.Map();

leftMap.setControlVisibility(true);
rightMap.setControlVisibility(true);

// SPLIT PANEL (Slider Effect)

var slider = ui.SplitPanel({
  firstPanel: leftMap,
  secondPanel: rightMap,
  wipe: true
});

// MAIN LAYOUT

var mainPanel = ui.Panel({
  layout: ui.Panel.Layout.Flow('horizontal'),
  style: {stretch: 'both'}
});

ui.root.add(mainPanel);
mainPanel.add(slider);


var sidePanel = ui.Panel({
  style: { width: '330px', padding: '12px' }
});
mainPanel.add(sidePanel);

// STUDY AREA

var bangladeshStates = ee.FeatureCollection('FAO/GAUL/2015/level1');

var dhaka = bangladeshStates.filter(
  ee.Filter.eq('ADM1_NAME', 'Dhaka')
);

leftMap.centerObject(dhaka, 7);
rightMap.centerObject(dhaka, 7);

leftMap.addLayer(dhaka, {color: 'red'}, 'Dhaka Division');
rightMap.addLayer(dhaka, {color: 'red'}, 'Dhaka Division');

/*********************************
 * SENTINEL-2 COLLECTION
 *********************************/
var s2 = ee.ImageCollection('COPERNICUS/S2_SR_HARMONIZED');

/*********************************
 * COMPOSITE FUNCTION
 *********************************/
function getComposite(start, end, cloud) {
  return s2
    .filterBounds(dhaka)
    .filterDate(start, end)
    .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', cloud))
    .select('B.*')
    .median()
    .clip(dhaka);
}

/*********************************
 * YEARLY COMPOSITES
 *********************************/
var composites = {
  '2017': getComposite('2017-01-01', '2017-12-31', 20),
  '2020': getComposite('2020-01-01', '2020-12-31', 5),
  '2023': getComposite('2023-01-01', '2023-02-10', 5)
};

/*********************************
 * TRAINING DATA (UNCHANGED)
 *********************************/
var training = Settlements
  .merge(Vegetation)
  .merge(Bareland)
  .merge(water);

/*********************************
 * SAMPLE REGIONS
 *********************************/
var trainingSamples = composites['2023'].sampleRegions({
  collection: training,
  properties: ['Class'],
  scale: 10
});

/*********************************
 * RANDOM FOREST CLASSIFIER
 *********************************/
var classifier = ee.Classifier.smileRandomForest(50).train({
  features: trainingSamples,
  classProperty: 'Class',
  inputProperties: composites['2023'].bandNames()
});

//CLASSIFICATION FUNCTION
 
function classify(img) {
  return img.classify(classifier);
}

//TRUE CLASSIFIED IMAGES
 
var classified = {
  '2017': classify(composites['2017']),
  '2020': classify(composites['2020']),
  '2023': classify(composites['2023'])
};

// VISUALIZATION SAFE (OPTION B)

var classifiedVisual = {
  '2017': classified['2017'].unmask(classified['2020']), // Fill gaps for display
  '2020': classified['2020'],
  '2023': classified['2023']
};

// VISUALIZATION PARAMETERS

var lulcVis = {
  min: 0,
  max: 3,
  palette: ['red', 'green', 'yellow', 'blue']
};

/*********************************
 * CHANGE DETECTION FUNCTION (GAP-FILLED 2017)
 *********************************/
function getChangeWithGapFill(y1, y2) {
  var c1 = classified[y1];
  if (y1 === '2017') {
    c1 = c1.unmask(classified['2020']);
  }

  var c1Remap = c1.unmask(-1).remap([0,1,2,3],[1,2,3,4]);
  var c2Remap = classified[y2].unmask(-1).remap([0,1,2,3],[1,2,3,4]);

  return c2Remap.subtract(c1Remap).neq(0);
}

// SIDE PANEL CONTENT

sidePanel.add(ui.Label({
  value: 'Interactive Land Use and Land Cover (LULC) Mapping \n of Dhaka Division',
  style: {fontSize: '16px', fontWeight: 'bold'}
}));

sidePanel.add(ui.Label(
  'This interactive Map visualizes Land Use and Land Cover (LULC) dynamics in Dhaka Division using Sentinel-2 imagery and Random Forest classification. Users can explore yearly LULC patterns and compare changes between selected years (2017 - 2023)'
));
sidePanel.add(ui.Label('Select Years', {fontWeight: 'bold'}));

var yearLeft = ui.Select({
  items: ['2017', '2020', '2023'],
  value: '2017'
});

var yearRight = ui.Select({
  items: ['2017', '2020', '2023'],
  value: '2023'
});

sidePanel.add(ui.Label('Left Map (Earlier Year)'));
sidePanel.add(yearLeft);

sidePanel.add(ui.Label('Right Map (Later Year)'));
sidePanel.add(yearRight);

var showChange = ui.Checkbox('Show Change Detection', false);
sidePanel.add(showChange);

// UPDATE MAPS FUNCTION

function updateMaps() {
  leftMap.layers().reset();
  rightMap.layers().reset();

  var y1 = yearLeft.getValue();
  var y2 = yearRight.getValue();

  // Visualize LULC
  leftMap.addLayer(
    classifiedVisual[y1],
    lulcVis,
    'LULC ' + y1
  );

  rightMap.addLayer(
    classifiedVisual[y2],
    lulcVis,
    'LULC ' + y2
  );

  // Change Detection
  if (showChange.getValue()) {
    var change = getChangeWithGapFill(y1, y2);
    rightMap.addLayer(
      change,
      {palette: ['white', 'red']},
      'Change Detection'
    );
  }
}

yearLeft.onChange(updateMaps);
yearRight.onChange(updateMaps);
showChange.onChange(updateMaps);

updateMaps();

// LULC LEGEND (LEFT MAP) ALIGNED + SPACED

var legend = ui.Panel({
  style: { position: 'bottom-left', padding: '12px', backgroundColor: 'white' } // slightly more padding
});

legend.add(ui.Label('LULC Classes', {fontWeight: 'bold', fontSize: '16px'})); // bigger title

var labels = ['Settlements', 'Vegetation', 'Bareland', 'Water'];
var colors = ['red', 'green', 'yellow', 'blue'];

labels.forEach(function(label, i) {
  var colorBox = ui.Label({
    style: {
      backgroundColor: colors[i],
      padding: '12px',      // bigger color box
      margin: '0',
      stretch: 'vertical'
    }
  });

  var desc = ui.Label({
    value: label,
    style: {
      margin: '0 0 0 8px', // space between color box and label
      stretch: 'vertical',
      fontSize: '14px',     // bigger text
      fontWeight: 'bold'
    }
  });

  var row = ui.Panel({
    widgets: [colorBox, desc],
    layout: ui.Panel.Layout.Flow('horizontal'),
    style: {margin: '0 0 6px 0'} // vertical spacing between rows
  });

  legend.add(row);
});

leftMap.add(legend);


/*********************************
 * CHANGE DETECTION LEGEND (RIGHT MAP)
 * ALIGNED + SPACED + BORDER FOR WHITE
 *********************************/
var changeLegend = ui.Panel({
  style: { position: 'bottom-right', padding: '12px', backgroundColor: 'white' } // more padding
});

changeLegend.add(ui.Label('Change Detection', {fontWeight: 'bold', fontSize: '16px'})); // bigger title

['No Change','Change'].forEach(function(label,i){
  var colorBoxStyle = {
    backgroundColor: i===0 ? 'white':'red', 
    padding:'12px',        // bigger color box
    margin:'0', 
    stretch:'vertical'
  };
  
  // Add border for white color
  if(i===0) {
    colorBoxStyle.border = '1px solid black';
  }

  var colorBox = ui.Label({style: colorBoxStyle});

  var desc = ui.Label({
    value: label,
    style: {
      margin:'0 0 0 8px',  // space between color box and text
      stretch:'vertical',
      fontSize: '14px',     // bigger text
      fontWeight: 'bold'
    }
  });

  var row = ui.Panel({
    widgets:[colorBox, desc],
    layout: ui.Panel.Layout.Flow('horizontal'),
    style: {margin: '0 0 6px 0'} // vertical spacing between rows
  });

  changeLegend.add(row);
});

rightMap.add(changeLegend);
