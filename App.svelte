<script>
  import { scaleSequential, scaleThreshold,scaleLinear, scaleBand, scaleTime } from 'd3-scale'
  import { schemeOranges } from 'd3-scale-chromatic'
  import { hexGrid } from './hex_grid.js'
  import { 
    Graphic, 
    PolygonLayer, 
    createGeoScales ,
    Section,
    RectangleLayer,
    Line,
    XAxis,
    YAxis 
    } from '@snlab/florence'
  import DataContainer from '@snlab/florence-datacontainer'

  // load data
  const hex = new DataContainer(hexGrid)
  const hexWithoutNull = hex.filter(row => row.mean_price !== null)
  // set up scales

  // 1. position
  const myGeoScale = createGeoScales(hex.domain('$geometry'))

  // 2. class breaks
  const numClasses = 6
  const classBins = hexWithoutNull
    .bin({ groupBy: 'mean_price', method: 'EqualInterval', numClasses: numClasses })
    .column('bins')
  console.log(hexWithoutNull)
  const classThresholds = binsToThreshold(classBins)

  function binsToThreshold (bins) {
    const thresholds = []
    for (let index = 1; index < bins.length; index++) {
      const bin = bins[index]
      thresholds.push(bin[0])
    }
    return thresholds
  }
  console.log(classBins, classThresholds)
  // 2. fill color
  const myColorScale = scaleThreshold()
    .domain(classThresholds)
    .range(schemeOranges[numClasses])
  console.log(myColorScale)
  const padding = { left: 30, bottom: 30, top: 0, right: 10 };
  const color = "rgb(93, 134, 156)";
  const color1 = "rgb(93, 250, 156)";


  // new data

  let salesDistribution = hexWithoutNull
    .bin({ groupBy: "mean_floor_area", method: "EqualInterval", numClasses: 500  })
    .summarise({ mean_price: { mean_price: "min" } })
  console.log("this is my aim" , salesDistribution._data.$key);

  let colors = [];
  for(let p in salesDistribution.column('mean_price')) {
    colors.push('rgb(93, 134, 156)');
  }

  salesDistribution.addColumn('color', colors);
// change color
let showid;
let color_change = "";
let change_colors = [];
  function ShowData(e){
    
    salesDistribution = hexWithoutNull
    .bin({ groupBy: "mean_floor_area", method: "EqualInterval", numClasses: 500  })
    .summarise({ mean_price: { mean_price: "min" } })

    change_colors = [];
    //console.log(hexWithoutNull._data.hex_id[e.key])
    showid = e.key
    console.log("show id is " , showid)
    for(let change in salesDistribution._data.$key) {
      if(change == showid){
        color_change = "#ff0000";
      }
      else{
        color_change = "rgb(93, 134, 156)";
      }
      change_colors.push(color_change);
    }
    
    salesDistribution.addColumn('color', change_colors);
    console.log(salesDistribution.column('color'));
  }

  function MoveData(e){
    salesDistribution = hexWithoutNull
    .bin({ groupBy: "mean_floor_area", method: "EqualInterval", numClasses: 500  })
    .summarise({ mean_price: { mean_price: "min" } })
 

    let colors = [];
    for(let p in salesDistribution.column('mean_price')) {
      colors.push('rgb(93, 134, 156)');
    }

  salesDistribution.addColumn('color', colors);
  }
  
</script>


<style>
  .hexgan-style{
    text-align: center;
  }
</style>


<div class="graph">
  <div class="main-chart">
    <div class="hexgan-style">
      <Graphic {...myGeoScale} flipY>
        <PolygonLayer
          onMouseover={(e)=> { ShowData(e) ;} }
          onMouseout = {(e)=> { MoveData(e) ;} }
          
          geometry={hex.column('$geometry')}
          stroke={'white'}
          strokeWidth={1} 
          fill={'#E0E0E0'}
        />
        <PolygonLayer 
          geometry={hexWithoutNull.column('$geometry')}
          stroke={'white'}
          strokeWidth={1} 
          fill={hexWithoutNull.map('mean_price', myColorScale)}
        />
      </Graphic>
    </div>
    <div></div>
    
    <Graphic width={2000} height={425}>
      <Section
        x1={0}
        x2={2000}
        y1={0}
        y2={400}
        {padding}
        flipY
        scaleX={scaleLinear().domain(salesDistribution.domain('bins'))}
        scaleY={scaleLinear().domain([
          0,
          salesDistribution.domain('mean_price')[1]
        ])}>
        
        <RectangleLayer
          onMouseover={(e)=> { ShowData(e) ;} }
          onMouseout = {(e)=> { MoveData(e) ;} }
          x1={salesDistribution.map('bins', bin => bin[0])}
          x2={salesDistribution.map('bins', bin => bin[1])}
          y1={0}
          y2={salesDistribution.column('mean_price')}
          fill={salesDistribution.column('color')} />
        
        <XAxis tickCount={5} labelFontSize={8} title="mean_floor_area" />
        <YAxis labelFontSize={8} title="mean price" />
      </Section>
    </Graphic>
  </div>
</div>