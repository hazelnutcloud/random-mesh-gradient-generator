<script context="module">
	
	//colors generated from coolors.co
	const hexColors = ["ffadad","ffd6a5","fdffb6","caffbf","9bf6ff","a0c4ff","bdb2ff","ffc6ff","fffffc"]
	
	function hexToRGB(hex) {
		const RGB = hex.match(/[0-9a-f]{1,2}/ig);
		return RGB.map(value => parseInt(value, 16));
	}
	
	export const colors = hexColors.map(hexToRGB);
</script>