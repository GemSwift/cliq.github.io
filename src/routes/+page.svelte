<script>
	let r=200;
	let cntSlices=9;
	let content=[
		"purple",
		"tomato",
		"bisque",
		"yellow",
		"green",
		"skyblue",
		"plum",
		"aqua",
		"white",
	];
	let rotation=0;
	let rotationInc=0;
	let contentRadius = r*4/5;
	let spinning = false;
	let outerWidth=17;
	
	$: sliceAngle=3.142*2/cntSlices;
	$: sliceDegrees=360/cntSlices|0;
	$: result=(((270-rotation)/sliceDegrees)+cntSlices|0)%cntSlices;

	setInterval(()=>{
		if (rotationInc > 0.06) {
			rotation+=rotationInc;	
			rotationInc *= 0.99;
			if (rotation > 360) rotation-= 360;
		} else {
			if ( spinning ) {
				broadcastResult();
				spinning=false;
			}
		}
	},10);

	const  url = 'https://ntfy.sh/';
	const  topic='spinningWheelTopic';
	function broadcastResult() {
		fetch(url+topic, {
	    method: 'POST',
  	  body: 'The winner is '+content[result%content.length]+' 😀 at '+new Date().toISOString()
		});
	}

</script>

<!-- <button on:click={()=>{rotation+=sliceDegrees/2}}>Inc</button> -->
<button on:click={()=>{rotationInc=15+5*Math.random();spinning=true;}}>Spin</button><br>{rotation|0} {rotationInc*100|0} {content[result%content.length]} ({result})<br>

<svg viewbox="0 0 {r*2+outerWidth*2} {r*2+outerWidth*2}" width={r+outerWidth*2} height={r+outerWidth*2}>
	<g transform="translate({r+outerWidth} {r+outerWidth}) rotate({rotation}) translate({-r} {-r}) ">
	<circle r={r} cx={r} cy={r} fill="lightgray" stroke-width={outerWidth} stroke="red"/>
		{#each Array(cntSlices) as angle, idx}
			{@const x=Math.cos(sliceAngle*idx)}
			{@const y=Math.sin(sliceAngle*idx)}
			{@const x2=Math.cos(sliceAngle*(idx+1))}
			{@const y2=Math.sin(sliceAngle*(idx+1))}
			{@const tx=Math.cos(sliceAngle*(idx+.5))*contentRadius+r}
			{@const ty=Math.sin(sliceAngle*(idx+.5))*contentRadius+r}
			{@const contentRotation=90+(360/cntSlices*(idx+0.5))|0}
			{@const path = [
					`M ${x*r+r} ${y*r+r}`, // Move
					`A ${r} ${r} 0 0 1 ${x2*r+r} ${y2*r+r}`, // Arc   
					`L ${r} ${r}`, // Line
				].join(' ')}
			<path d={path} fill={content[idx%content.length]}/>}
			<g transform="translate({tx} {ty}) rotate({contentRotation}) translate({-tx} {-ty}) ">
				<text font-size={24} x={tx} y={ty} text-anchor ="middle">{content[idx%content.length]}</text>}
			</g>
		{/each}
	</g>
</svg>
<br>
Subscribers can see the result at <a href={url+topic} target="_blank">{url+topic}</a>