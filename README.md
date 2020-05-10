# Build Your Own Food Delivery Applications
* Download project data from the following link https://github.com/geo-codes-com/FoodDeliveryApp
* Open your IDE for example Visual Studio Code or Webstorm
* Create a new file and rename it to index.html and type the following code in index.html
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Food Delivery Application</title>
</head>
<body>
    
</body>
</html>
```
* In head section include HERE MAPs API and Booststrap library
```html
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.4.1/css/bootstrap.min.css">
<script src="https://ajax.googleapis.com/ajax/libs/jquery/3.4.1/jquery.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.16.0/umd/popper.min.js"></script>
<script src="https://maxcdn.bootstrapcdn.com/bootstrap/4.4.1/js/bootstrap.min.js"></script>
<script src="https://js.api.here.com/v3/3.1/mapsjs-core.js"></script>
<script src="https://js.api.here.com/v3/3.1/mapsjs-service.js"></script>
<script src="https://js.api.here.com/v3/3.1/mapsjs-ui.js"></script>
<link rel="stylesheet" href="https://js.api.here.com/v3/3.1/mapsjs-ui.css" />
<script src="https://js.api.here.com/v3/3.1/mapsjs-mapevents.js"></script>
```
* In body section add the following div element
```html
<nav class="navbar navbar-dark bg-primary">
	<a class="navbar-brand" href="#">Food Delivery</a>
</nav>
<div id="mapContainer"></div>
```

* After div element add the following style tag
```html
<style>
	html, body, #mapContainer {
		width: 100%; height: 100%;
	}
</style>
```
* After Style tag add the following script tag
```html
<script>

</script>
```
* Inside script tag add the following code
```javascript
var platform = new H.service.Platform({
	'apikey':'PUT YOUR API KEY HERE'
});
var layers = platform.createDefaultLayers();
var map = new H.Map(
	document.getElementById('mapContainer'),
	layers.vector.normal.map,
	{
		zoom: 16,
		center:{lat:21.422542, lng:39.826230}
	}
);
var ui = new H.ui.UI.createDefault(map,layers);
ui.getControl('mapsettings').setAlignment('top-left');
var mapEvents = new H.mapevents.MapEvents(map);
var behavior = new H.mapevents.Behavior(mapEvents);
```
* Add placeService and routingService instance
```javascript
var placesService = platform.getPlacesService();
var router = platform.getRoutingService();
```
* Type the following code to enable user to determine his location and find all restaurants around him
```javascript
var myLocation;
map.addEventListener('tap',addLoc);
function addLoc(evt){
	var coords = map.screenToGeo(
		evt.currentPointer.viewportX,
		evt.currentPointer.viewportY
	);
	myLocation = new H.map.Marker({lat: coords.lat,lng: coords.lng});
	map.addObject(myLocation);
	// show all food delivery restaurants around your location
	parameters = {'at':myLocation.b.lat+','+myLocation.b.lng,
		'cat':'eat-drink'};

	placesService.explore(parameters, function (results) {
			var restGroup = new H.map.Group();
			var items = results.results.items;
			items.forEach(function(item){
				let restIcon = new H.map.Icon('img/03.png',{size:{w:35,h:35}});
				let restPosition = {lat: item.position[0],lng:item.position[1]};
				let data = item.title;
				let restMarker = new H.map.Marker(restPosition,{icon: restIcon});
				restMarker.setData(data);
				restGroup.addObject(restMarker);
			});
			map.addObject(restGroup);
			map.getViewModel().setLookAtData({
					bounds: restGroup.getBoundingBox()
			});
	});
}
```
* Add The Following event to select a restaurant
```javascript
map.addEventListener('tap', function (evt) {
	if (evt.target instanceof H.map.Marker) {
		var bubble =  new H.ui.InfoBubble(evt.target.getGeometry(), {
		// read custom data
		content: evt.target.getData()
		});
		// show info bubble
		ui.addBubble(bubble);
		let restPos= evt.target.getGeometry();
		showRoute(restPos);
	}
});
```
* Add Following code to calculate the best route
```javascript
function showRoute(restPos){
	var routingParams = {
		'mode':'fastest;car',
		'waypoint0': restPos.lat+','+restPos.lng ,
		'waypoint1': myLocation.b.lat+','+myLocation.b.lng ,
		'representation':'display'
	};
	// calculate the best route
	router.calculateRoute(routingParams, function(result){
		var route, routeShape, startPoint;
		if(result.response.route){
			route = result.response.route[0];
			routeShape = route.shape;
			lineString = new H.geo.LineString();
			routeShape.forEach(function(point){
				var parts = point.split(',');
				lineString.pushLatLngAlt(parts[0],parts[1]);
			});
			var routeLine = new H.map.Polyline(lineString,{
				style:{strokeColor:'blue', lineWidth:10}
			});
			map.addObjects([routeLine]);
			map.getViewModel().setLookAtData({
				bounds: routeLine.getBoundingBox()
			});
		}
	});
}
```
