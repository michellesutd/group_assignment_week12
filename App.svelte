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
  const padding = { left: 90, bottom: 30, top: 0, right: 10 };
  const color = "rgb(93, 134, 156)";
  const color1 = "rgb(93, 250, 156)";


  // new data

  let salesDistribution = hexWithoutNull
    .bin({ groupBy: "mean_floor_area", method: "EqualInterval", numClasses: 200  })
    .summarise({ mean_price: { mean_price: "mean" } })
  //console.log("this is my aim" , salesDistribution)

let showid;
  function ShowData(e){
    //console.log(hexWithoutNull._data.hex_id[e.key])
    showid = hexWithoutNull._data.hex_id[e.key]
    console.log("show id is " , showid)
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
    
    <Graphic width={1800} height={425}>
      <Section
        x1={0}
        x2={1800}
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
          
          x1={salesDistribution.map('bins', bin => bin[0])}
          x2={salesDistribution.map('bins', bin => bin[1])}
          y1={0}
          y2={salesDistribution.column('mean_price')}
          fill={color} />
        
        <XAxis tickCount={5} labelFontSize={8} title="mean_floor_area" />
        <YAxis labelFontSize={8} title="mean price" />
      </Section>
    </Graphic>
  </div>
</div>