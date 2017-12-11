<template>

        <div class="row">

            <div class="col-md-3">
                <div class="row">
                    <b-card-group deckclass="mb-2">
                        <b-card bg-variant="dark" text-variant="white" header="Tools" class="text-center">
                            <b-container>
                                <div class="row">
                                    <b-button :pressed.sync="selectedDistance" v-on:click="show_distance" variant="primary">Calculate distance</b-button>
                                </div>
                                <hr/>
                                <div class="row">
                                    <b-button :pressed.sync="selectedDrawing" v-on:click="show_drawing" variant="primary">Draw shapes</b-button>
                                </div>
                                <hr/>
                                <div class="row">
                                    <b-button :pressed.sync="selectedCircle" v-on:click="show_circle" variant="primary">Draw circle</b-button>
                                    <b-form-input v-model.number="radius"
                                      type="number"
                                      placeholder="Radius"></b-form-input>
                                </div>
                                <hr/>
                                <div class="row">
                                    <b-button :disabled="!selectedCircle" v-on:click="show_contains" variant="primary">Contains</b-button>
                                </div>
                            </b-container>
                        </b-card>
                    </b-card-group>
                </div>
                <div class="row">
                    <b-list-group>
                        <b-list-group-item v-for="layer in layers">
                            {{ layer.name }}
                        </b-list-group-item>
                    </b-list-group>
                </div>
            </div>

            <div class="col-md-8">
                <div>
                    <b-navbar type="dark" variant="dark" toggleable>
                        <b-navbar-toggle target="nav_dropdown_collapse"></b-navbar-toggle>
                        <b-collapse is-nav id="nav_dropdown_collapse">
                          <b-navbar-nav>
                            <b-nav-form>
                              <b-form-input class="mr-sm-2" type="text" placeholder="file.kml" v-model="fileName"></b-form-input>
                              <b-button class="my-2 my-sm-0" type="submit" v-on:click="load_data(fileName)">Load</b-button>
                            </b-nav-form>
                            <!-- Navbar dropdowns -->
                            <b-nav-item-dropdown text="Style" right>
                              <b-dropdown-item v-on:click="change_style('standard')" >Standard</b-dropdown-item>
                              <b-dropdown-item v-on:click="change_style('silver')" >Silver</b-dropdown-item>
                              <b-dropdown-item v-on:click="change_style('retro')" >Retro</b-dropdown-item>
                              <b-dropdown-item v-on:click="change_style('dark')" >Dark</b-dropdown-item>
                              <b-dropdown-item v-on:click="change_style('night')" >Night</b-dropdown-item>
                              <b-dropdown-item v-on:click="change_style('augbergine')" >Aubergine</b-dropdown-item>
                            </b-nav-item-dropdown>
                          </b-navbar-nav>
                        </b-collapse>
                      </b-navbar>
                    </div>
                
                <div>

                    <div class="google-map":id="mapName"></div>

                    <div id="floating-panel" v-if="selectedDistance">
                      Origin: <input type="text" :value="marker1.getPosition()" readonly size="38">
                      Destination: <input type="text" :value="marker2.getPosition()" readonly size="38"><br>
                      Distance: <input type="text" :value="distance" readonly size="30"> meters
                    </div>

                    <b-card v-if="selectedCircle" bg-variant="dark"
                            text-variant="white"
                            header="Contained points"
                            class="text-center">
                        <b-row>
                            <b-col>Attribute</b-col>
                            <b-col>Latitude</b-col>
                            <b-col>Longitude</b-col>
                            <hr/>
                        </b-row>
                        <b-row v-for="item in contained_markers">
                            <b-col>{{item.title}}</b-col>
                            <b-col>{{item.lat}}</b-col>
                            <b-col>{{item.lng}}</b-col>
                        </b-row>
                    </b-card>

                </div>

            </div>

        </div>

</template>

<script>
export default {
  name: 'google-map',
  props: ['name'],
  data: function () {
    return {
      mapName: this.name + "-map",
      map: null,
      bounds: null,
      markers: [],
      contained_markers: [],
      fileName: '',
      selectedDistance: false,
      selectedDrawing: false,
      selectedCircle: false,
      distance: 0,
      drawingManager: null,
      all_shapes: [],
      marker1: null,
      marker2: null,
      poly: null,
      geodesicPoly: null,
      lattitud: 19.433941,
      longitude: -99.133563,
      circle: null,
      radius: 1000,
      layers : [],
      mapStyle: [
          {
            "elementType": "geometry",
            "stylers": [
              {
                "color": "#ebe3cd"
              }
            ]
          },
          {
            "elementType": "labels.text.fill",
            "stylers": [
              {
                "color": "#523735"
              }
            ]
          },
          {
            "elementType": "labels.text.stroke",
            "stylers": [
              {
                "color": "#f5f1e6"
              }
            ]
          },
          {
            "featureType": "administrative",
            "elementType": "geometry.stroke",
            "stylers": [
              {
                "color": "#c9b2a6"
              }
            ]
          },
          {
            "featureType": "administrative.land_parcel",
            "elementType": "geometry.stroke",
            "stylers": [
              {
                "color": "#dcd2be"
              }
            ]
          },
          {
            "featureType": "administrative.land_parcel",
            "elementType": "labels.text.fill",
            "stylers": [
              {
                "color": "#ae9e90"
              }
            ]
          },
          {
            "featureType": "landscape.natural",
            "elementType": "geometry",
            "stylers": [
              {
                "color": "#dfd2ae"
              }
            ]
          },
          {
            "featureType": "poi",
            "elementType": "geometry",
            "stylers": [
              {
                "color": "#dfd2ae"
              }
            ]
          },
          {
            "featureType": "poi",
            "elementType": "labels.text.fill",
            "stylers": [
              {
                "color": "#93817c"
              }
            ]
          },
          {
            "featureType": "poi.park",
            "elementType": "geometry.fill",
            "stylers": [
              {
                "color": "#a5b076"
              }
            ]
          },
          {
            "featureType": "poi.park",
            "elementType": "labels.text.fill",
            "stylers": [
              {
                "color": "#447530"
              }
            ]
          },
          {
            "featureType": "road",
            "elementType": "geometry",
            "stylers": [
              {
                "color": "#f5f1e6"
              }
            ]
          },
          {
            "featureType": "road.arterial",
            "elementType": "geometry",
            "stylers": [
              {
                "color": "#fdfcf8"
              }
            ]
          },
          {
            "featureType": "road.highway",
            "elementType": "geometry",
            "stylers": [
              {
                "color": "#f8c967"
              }
            ]
          },
          {
            "featureType": "road.highway",
            "elementType": "geometry.stroke",
            "stylers": [
              {
                "color": "#e9bc62"
              }
            ]
          },
          {
            "featureType": "road.highway.controlled_access",
            "elementType": "geometry",
            "stylers": [
              {
                "color": "#e98d58"
              }
            ]
          },
          {
            "featureType": "road.highway.controlled_access",
            "elementType": "geometry.stroke",
            "stylers": [
              {
                "color": "#db8555"
              }
            ]
          },
          {
            "featureType": "road.local",
            "elementType": "labels.text.fill",
            "stylers": [
              {
                "color": "#806b63"
              }
            ]
          },
          {
            "featureType": "transit.line",
            "elementType": "geometry",
            "stylers": [
              {
                "color": "#dfd2ae"
              }
            ]
          },
          {
            "featureType": "transit.line",
            "elementType": "labels.text.fill",
            "stylers": [
              {
                "color": "#8f7d77"
              }
            ]
          },
          {
            "featureType": "transit.line",
            "elementType": "labels.text.stroke",
            "stylers": [
              {
                "color": "#ebe3cd"
              }
            ]
          },
          {
            "featureType": "transit.station",
            "elementType": "geometry",
            "stylers": [
              {
                "color": "#dfd2ae"
              }
            ]
          },
          {
            "featureType": "water",
            "elementType": "geometry.fill",
            "stylers": [
              {
                "color": "#b9d3c2"
              }
            ]
          },
          {
            "featureType": "water",
            "elementType": "labels.text.fill",
            "stylers": [
              {
                "color": "#92998d"
              }
            ]
          }
        ]
    }
  },

  methods: {

        
        update() {
              var path = [this.marker1.getPosition(), this.marker2.getPosition()];
              this.geodesicPoly.setPath(path);
              this.distance = google.maps.geometry.spherical.computeDistanceBetween(path[0], path[1]);
        },

        initMap(){
            const element = document.getElementById(this.mapName)
            const options = {
              zoom: 12,
              center: {lat: this.lattitud, lng: this.longitude},
              styles: this.mapStyle
            }
            
            this.map = new google.maps.Map(element, options);

            const vm = this;

            $.ajax({
              url: 'https://gist.githubusercontent.com/IsaacRodgz/c4882dad5b1e7fe3f4212fc2ef06b744/raw/732ece85c6016fea41301dbf33efa29612b9cb49/points.json',
              method: 'GET', context: this
            }).then(function(data) {
                var myJson = $.parseJSON(data);
                for (var index = 0; index < myJson.data.length; ++index) {
                    var marker = new google.maps.Marker({
                        position: {lat: myJson.data[index].lat, lng: myJson.data[index].lng},
                        title: myJson.data[index].text,
                        map: vm.map
                    });
                    vm.markers.push(marker);
                }
            });
            
        },

        load_data(dir){
            if(dir == ''){
                console.log("Directorio invalido");
            }
            else{
                
                var ctaLayer = new google.maps.KmlLayer({
                    url: dir,
                    map: this.map
                });
                this.fileName = '';
                this.layers.push({"name":dir.split("/").slice(-1)[0]});
            }
        },

        show_distance(){
            if(this.selectedDistance){
                this.marker1 = new google.maps.Marker({
                    map: this.map,
                    draggable: true,
                    position: {lat: 19.368884, lng: -99.206227}
                });

                this.marker2 = new google.maps.Marker({
                    map: this.map,
                    draggable: true,
                    position: {lat: 19.378600, lng: -99.027356}
                });

                var bounds = new google.maps.LatLngBounds(this.marker1.getPosition(), this.marker2.getPosition());
                this.map.fitBounds(bounds);

                google.maps.event.addListener(this.marker1, 'position_changed', this.update);
                google.maps.event.addListener(this.marker2, 'position_changed', this.update);

                this.geodesicPoly = new google.maps.Polyline({
                    strokeColor: '#CC0099',
                    strokeOpacity: 1.0,
                    strokeWeight: 3,
                    geodesic: true,
                    map: this.map
                });

                this.update();
            }
            else{
                this.marker1.setMap(null);
                this.marker2.setMap(null);
                this.geodesicPoly.setMap(null);
            }
        },

        show_drawing(){
            if(this.selectedDrawing){
                this.drawingManager = new google.maps.drawing.DrawingManager(
                    {
                        drawingMode: google.maps.drawing.OverlayType.MARKER,
                        drawingControl: true,
                        drawingControlOptions:
                        {
                          position: google.maps.ControlPosition.TOP_CENTER,
                          drawingModes: ['marker', 'circle', 'polygon', 'polyline', 'rectangle']
                        },
                        markerOptions: {icon: 'https://developers.google.com/maps/documentation/javascript/examples/full/images/beachflag.png'}
                    }
                );
                
                this.drawingManager.setMap(this.map);

                google.maps.event.addListener(this.drawingManager, 'overlaycomplete', function(event) {
                  this.all_shapes.push(event);
                  if (event.type == 'circle') {
                    var radius = event.overlay.getRadius();
                    console.log(radius);
                  }
                }.bind(this));
            }
            else{
                for (var i=0; i < this.all_shapes.length; i++)
                {
                    this.all_shapes[i].overlay.setMap(null);
                }
                this.all_shapes = [];
                this.drawingManager.setMap(null);
                google.maps.event.clearListeners(this.drawingManager, 'overlaycomplete');
            }
        },

        show_circle(){
            if(this.selectedCircle){
                this.circle = new google.maps.Circle({
                    strokeColor: '#FF0000',
                    strokeOpacity: 0.8,
                    strokeWeight: 2,
                    fillColor: '#FF0000',
                    fillOpacity: 0.35,
                    map: this.map,
                    center: {lat: this.lattitud, lng: this.longitude},
                    radius: this.radius,
                    draggable: true
                });
            }
            else{
                this.circle.setMap(null);
            }   
        },

        show_contains(){
            if(this.selectedCircle){
                for (var i=0; i < this.markers.length; i++){
                    var contained = this.circle.getBounds().contains(this.markers[i].getPosition())
                    if(contained){
                        var lat = this.markers[i].getPosition().lat();
                        var lng = this.markers[i].getPosition().lng();
                        this.contained_markers.push({"title": this.markers[i].getTitle(), "lat": lat, "lng": lng});                   
                    }
                }
            }
        },
        
        change_style(s){
            if(s === "standard"){
                this.mapStyle = []   
            }
            else if(s === "silver"){
                this.mapStyle = [
                  {
                    "elementType": "geometry",
                    "stylers": [
                      {
                        "color": "#f5f5f5"
                      }
                    ]
                  },
                  {
                    "elementType": "labels.icon",
                    "stylers": [
                      {
                        "visibility": "off"
                      }
                    ]
                  },
                  {
                    "elementType": "labels.text.fill",
                    "stylers": [
                      {
                        "color": "#616161"
                      }
                    ]
                  },
                  {
                    "elementType": "labels.text.stroke",
                    "stylers": [
                      {
                        "color": "#f5f5f5"
                      }
                    ]
                  },
                  {
                    "featureType": "administrative.land_parcel",
                    "elementType": "labels.text.fill",
                    "stylers": [
                      {
                        "color": "#bdbdbd"
                      }
                    ]
                  },
                  {
                    "featureType": "poi",
                    "elementType": "geometry",
                    "stylers": [
                      {
                        "color": "#eeeeee"
                      }
                    ]
                  },
                  {
                    "featureType": "poi",
                    "elementType": "labels.text.fill",
                    "stylers": [
                      {
                        "color": "#757575"
                      }
                    ]
                  },
                  {
                    "featureType": "poi.park",
                    "elementType": "geometry",
                    "stylers": [
                      {
                        "color": "#e5e5e5"
                      }
                    ]
                  },
                  {
                    "featureType": "poi.park",
                    "elementType": "labels.text.fill",
                    "stylers": [
                      {
                        "color": "#9e9e9e"
                      }
                    ]
                  },
                  {
                    "featureType": "road",
                    "elementType": "geometry",
                    "stylers": [
                      {
                        "color": "#ffffff"
                      }
                    ]
                  },
                  {
                    "featureType": "road.arterial",
                    "elementType": "labels.text.fill",
                    "stylers": [
                      {
                        "color": "#757575"
                      }
                    ]
                  },
                  {
                    "featureType": "road.highway",
                    "elementType": "geometry",
                    "stylers": [
                      {
                        "color": "#dadada"
                      }
                    ]
                  },
                  {
                    "featureType": "road.highway",
                    "elementType": "labels.text.fill",
                    "stylers": [
                      {
                        "color": "#616161"
                      }
                    ]
                  },
                  {
                    "featureType": "road.local",
                    "elementType": "labels.text.fill",
                    "stylers": [
                      {
                        "color": "#9e9e9e"
                      }
                    ]
                  },
                  {
                    "featureType": "transit.line",
                    "elementType": "geometry",
                    "stylers": [
                      {
                        "color": "#e5e5e5"
                      }
                    ]
                  },
                  {
                    "featureType": "transit.station",
                    "elementType": "geometry",
                    "stylers": [
                      {
                        "color": "#eeeeee"
                      }
                    ]
                  },
                  {
                    "featureType": "water",
                    "elementType": "geometry",
                    "stylers": [
                      {
                        "color": "#c9c9c9"
                      }
                    ]
                  },
                  {
                    "featureType": "water",
                    "elementType": "labels.text.fill",
                    "stylers": [
                      {
                        "color": "#9e9e9e"
                      }
                    ]
                  }
                ]
            }
            else if(s === "retro"){
                this.mapStyle = [
                  {
                    "elementType": "geometry",
                    "stylers": [
                      {
                        "color": "#ebe3cd"
                      }
                    ]
                  },
                  {
                    "elementType": "labels.text.fill",
                    "stylers": [
                      {
                        "color": "#523735"
                      }
                    ]
                  },
                  {
                    "elementType": "labels.text.stroke",
                    "stylers": [
                      {
                        "color": "#f5f1e6"
                      }
                    ]
                  },
                  {
                    "featureType": "administrative",
                    "elementType": "geometry.stroke",
                    "stylers": [
                      {
                        "color": "#c9b2a6"
                      }
                    ]
                  },
                  {
                    "featureType": "administrative.land_parcel",
                    "elementType": "geometry.stroke",
                    "stylers": [
                      {
                        "color": "#dcd2be"
                      }
                    ]
                  },
                  {
                    "featureType": "administrative.land_parcel",
                    "elementType": "labels.text.fill",
                    "stylers": [
                      {
                        "color": "#ae9e90"
                      }
                    ]
                  },
                  {
                    "featureType": "landscape.natural",
                    "elementType": "geometry",
                    "stylers": [
                      {
                        "color": "#dfd2ae"
                      }
                    ]
                  },
                  {
                    "featureType": "poi",
                    "elementType": "geometry",
                    "stylers": [
                      {
                        "color": "#dfd2ae"
                      }
                    ]
                  },
                  {
                    "featureType": "poi",
                    "elementType": "labels.text.fill",
                    "stylers": [
                      {
                        "color": "#93817c"
                      }
                    ]
                  },
                  {
                    "featureType": "poi.park",
                    "elementType": "geometry.fill",
                    "stylers": [
                      {
                        "color": "#a5b076"
                      }
                    ]
                  },
                  {
                    "featureType": "poi.park",
                    "elementType": "labels.text.fill",
                    "stylers": [
                      {
                        "color": "#447530"
                      }
                    ]
                  },
                  {
                    "featureType": "road",
                    "elementType": "geometry",
                    "stylers": [
                      {
                        "color": "#f5f1e6"
                      }
                    ]
                  },
                  {
                    "featureType": "road.arterial",
                    "elementType": "geometry",
                    "stylers": [
                      {
                        "color": "#fdfcf8"
                      }
                    ]
                  },
                  {
                    "featureType": "road.highway",
                    "elementType": "geometry",
                    "stylers": [
                      {
                        "color": "#f8c967"
                      }
                    ]
                  },
                  {
                    "featureType": "road.highway",
                    "elementType": "geometry.stroke",
                    "stylers": [
                      {
                        "color": "#e9bc62"
                      }
                    ]
                  },
                  {
                    "featureType": "road.highway.controlled_access",
                    "elementType": "geometry",
                    "stylers": [
                      {
                        "color": "#e98d58"
                      }
                    ]
                  },
                  {
                    "featureType": "road.highway.controlled_access",
                    "elementType": "geometry.stroke",
                    "stylers": [
                      {
                        "color": "#db8555"
                      }
                    ]
                  },
                  {
                    "featureType": "road.local",
                    "elementType": "labels.text.fill",
                    "stylers": [
                      {
                        "color": "#806b63"
                      }
                    ]
                  },
                  {
                    "featureType": "transit.line",
                    "elementType": "geometry",
                    "stylers": [
                      {
                        "color": "#dfd2ae"
                      }
                    ]
                  },
                  {
                    "featureType": "transit.line",
                    "elementType": "labels.text.fill",
                    "stylers": [
                      {
                        "color": "#8f7d77"
                      }
                    ]
                  },
                  {
                    "featureType": "transit.line",
                    "elementType": "labels.text.stroke",
                    "stylers": [
                      {
                        "color": "#ebe3cd"
                      }
                    ]
                  },
                  {
                    "featureType": "transit.station",
                    "elementType": "geometry",
                    "stylers": [
                      {
                        "color": "#dfd2ae"
                      }
                    ]
                  },
                  {
                    "featureType": "water",
                    "elementType": "geometry.fill",
                    "stylers": [
                      {
                        "color": "#b9d3c2"
                      }
                    ]
                  },
                  {
                    "featureType": "water",
                    "elementType": "labels.text.fill",
                    "stylers": [
                      {
                        "color": "#92998d"
                      }
                    ]
                  }
                ]
            }
            else if(s === "night"){
                this.mapStyle = [
                  {
                    "elementType": "geometry",
                    "stylers": [
                      {
                        "color": "#242f3e"
                      }
                    ]
                  },
                  {
                    "elementType": "labels.text.fill",
                    "stylers": [
                      {
                        "color": "#746855"
                      }
                    ]
                  },
                  {
                    "elementType": "labels.text.stroke",
                    "stylers": [
                      {
                        "color": "#242f3e"
                      }
                    ]
                  },
                  {
                    "featureType": "administrative.locality",
                    "elementType": "labels.text.fill",
                    "stylers": [
                      {
                        "color": "#d59563"
                      }
                    ]
                  },
                  {
                    "featureType": "poi",
                    "elementType": "labels.text.fill",
                    "stylers": [
                      {
                        "color": "#d59563"
                      }
                    ]
                  },
                  {
                    "featureType": "poi.park",
                    "elementType": "geometry",
                    "stylers": [
                      {
                        "color": "#263c3f"
                      }
                    ]
                  },
                  {
                    "featureType": "poi.park",
                    "elementType": "labels.text.fill",
                    "stylers": [
                      {
                        "color": "#6b9a76"
                      }
                    ]
                  },
                  {
                    "featureType": "road",
                    "elementType": "geometry",
                    "stylers": [
                      {
                        "color": "#38414e"
                      }
                    ]
                  },
                  {
                    "featureType": "road",
                    "elementType": "geometry.stroke",
                    "stylers": [
                      {
                        "color": "#212a37"
                      }
                    ]
                  },
                  {
                    "featureType": "road",
                    "elementType": "labels.text.fill",
                    "stylers": [
                      {
                        "color": "#9ca5b3"
                      }
                    ]
                  },
                  {
                    "featureType": "road.highway",
                    "elementType": "geometry",
                    "stylers": [
                      {
                        "color": "#746855"
                      }
                    ]
                  },
                  {
                    "featureType": "road.highway",
                    "elementType": "geometry.stroke",
                    "stylers": [
                      {
                        "color": "#1f2835"
                      }
                    ]
                  },
                  {
                    "featureType": "road.highway",
                    "elementType": "labels.text.fill",
                    "stylers": [
                      {
                        "color": "#f3d19c"
                      }
                    ]
                  },
                  {
                    "featureType": "transit",
                    "elementType": "geometry",
                    "stylers": [
                      {
                        "color": "#2f3948"
                      }
                    ]
                  },
                  {
                    "featureType": "transit.station",
                    "elementType": "labels.text.fill",
                    "stylers": [
                      {
                        "color": "#d59563"
                      }
                    ]
                  },
                  {
                    "featureType": "water",
                    "elementType": "geometry",
                    "stylers": [
                      {
                        "color": "#17263c"
                      }
                    ]
                  },
                  {
                    "featureType": "water",
                    "elementType": "labels.text.fill",
                    "stylers": [
                      {
                        "color": "#515c6d"
                      }
                    ]
                  },
                  {
                    "featureType": "water",
                    "elementType": "labels.text.stroke",
                    "stylers": [
                      {
                        "color": "#17263c"
                      }
                    ]
                  }
                ]
            }
            else if(s === "augbergine"){
                this.mapStyle = [
                  {
                    "elementType": "geometry",
                    "stylers": [
                      {
                        "color": "#1d2c4d"
                      }
                    ]
                  },
                  {
                    "elementType": "labels.text.fill",
                    "stylers": [
                      {
                        "color": "#8ec3b9"
                      }
                    ]
                  },
                  {
                    "elementType": "labels.text.stroke",
                    "stylers": [
                      {
                        "color": "#1a3646"
                      }
                    ]
                  },
                  {
                    "featureType": "administrative.country",
                    "elementType": "geometry.stroke",
                    "stylers": [
                      {
                        "color": "#4b6878"
                      }
                    ]
                  },
                  {
                    "featureType": "administrative.land_parcel",
                    "elementType": "labels.text.fill",
                    "stylers": [
                      {
                        "color": "#64779e"
                      }
                    ]
                  },
                  {
                    "featureType": "administrative.province",
                    "elementType": "geometry.stroke",
                    "stylers": [
                      {
                        "color": "#4b6878"
                      }
                    ]
                  },
                  {
                    "featureType": "landscape.man_made",
                    "elementType": "geometry.stroke",
                    "stylers": [
                      {
                        "color": "#334e87"
                      }
                    ]
                  },
                  {
                    "featureType": "landscape.natural",
                    "elementType": "geometry",
                    "stylers": [
                      {
                        "color": "#023e58"
                      }
                    ]
                  },
                  {
                    "featureType": "poi",
                    "elementType": "geometry",
                    "stylers": [
                      {
                        "color": "#283d6a"
                      }
                    ]
                  },
                  {
                    "featureType": "poi",
                    "elementType": "labels.text.fill",
                    "stylers": [
                      {
                        "color": "#6f9ba5"
                      }
                    ]
                  },
                  {
                    "featureType": "poi",
                    "elementType": "labels.text.stroke",
                    "stylers": [
                      {
                        "color": "#1d2c4d"
                      }
                    ]
                  },
                  {
                    "featureType": "poi.park",
                    "elementType": "geometry.fill",
                    "stylers": [
                      {
                        "color": "#023e58"
                      }
                    ]
                  },
                  {
                    "featureType": "poi.park",
                    "elementType": "labels.text.fill",
                    "stylers": [
                      {
                        "color": "#3C7680"
                      }
                    ]
                  },
                  {
                    "featureType": "road",
                    "elementType": "geometry",
                    "stylers": [
                      {
                        "color": "#304a7d"
                      }
                    ]
                  },
                  {
                    "featureType": "road",
                    "elementType": "labels.text.fill",
                    "stylers": [
                      {
                        "color": "#98a5be"
                      }
                    ]
                  },
                  {
                    "featureType": "road",
                    "elementType": "labels.text.stroke",
                    "stylers": [
                      {
                        "color": "#1d2c4d"
                      }
                    ]
                  },
                  {
                    "featureType": "road.highway",
                    "elementType": "geometry",
                    "stylers": [
                      {
                        "color": "#2c6675"
                      }
                    ]
                  },
                  {
                    "featureType": "road.highway",
                    "elementType": "geometry.stroke",
                    "stylers": [
                      {
                        "color": "#255763"
                      }
                    ]
                  },
                  {
                    "featureType": "road.highway",
                    "elementType": "labels.text.fill",
                    "stylers": [
                      {
                        "color": "#b0d5ce"
                      }
                    ]
                  },
                  {
                    "featureType": "road.highway",
                    "elementType": "labels.text.stroke",
                    "stylers": [
                      {
                        "color": "#023e58"
                      }
                    ]
                  },
                  {
                    "featureType": "transit",
                    "elementType": "labels.text.fill",
                    "stylers": [
                      {
                        "color": "#98a5be"
                      }
                    ]
                  },
                  {
                    "featureType": "transit",
                    "elementType": "labels.text.stroke",
                    "stylers": [
                      {
                        "color": "#1d2c4d"
                      }
                    ]
                  },
                  {
                    "featureType": "transit.line",
                    "elementType": "geometry.fill",
                    "stylers": [
                      {
                        "color": "#283d6a"
                      }
                    ]
                  },
                  {
                    "featureType": "transit.station",
                    "elementType": "geometry",
                    "stylers": [
                      {
                        "color": "#3a4762"
                      }
                    ]
                  },
                  {
                    "featureType": "water",
                    "elementType": "geometry",
                    "stylers": [
                      {
                        "color": "#0e1626"
                      }
                    ]
                  },
                  {
                    "featureType": "water",
                    "elementType": "labels.text.fill",
                    "stylers": [
                      {
                        "color": "#4e6d70"
                      }
                    ]
                  }
                ]
            }
            else{
                this.mapStyle = [
                  {
                    "elementType": "geometry",
                    "stylers": [
                      {
                        "color": "#212121"
                      }
                    ]
                  },
                  {
                    "elementType": "labels.icon",
                    "stylers": [
                      {
                        "visibility": "off"
                      }
                    ]
                  },
                  {
                    "elementType": "labels.text.fill",
                    "stylers": [
                      {
                        "color": "#757575"
                      }
                    ]
                  },
                  {
                    "elementType": "labels.text.stroke",
                    "stylers": [
                      {
                        "color": "#212121"
                      }
                    ]
                  },
                  {
                    "featureType": "administrative",
                    "elementType": "geometry",
                    "stylers": [
                      {
                        "color": "#757575"
                      }
                    ]
                  },
                  {
                    "featureType": "administrative.country",
                    "elementType": "labels.text.fill",
                    "stylers": [
                      {
                        "color": "#9e9e9e"
                      }
                    ]
                  },
                  {
                    "featureType": "administrative.land_parcel",
                    "stylers": [
                      {
                        "visibility": "off"
                      }
                    ]
                  },
                  {
                    "featureType": "administrative.locality",
                    "elementType": "labels.text.fill",
                    "stylers": [
                      {
                        "color": "#bdbdbd"
                      }
                    ]
                  },
                  {
                    "featureType": "poi",
                    "elementType": "labels.text.fill",
                    "stylers": [
                      {
                        "color": "#757575"
                      }
                    ]
                  },
                  {
                    "featureType": "poi.park",
                    "elementType": "geometry",
                    "stylers": [
                      {
                        "color": "#181818"
                      }
                    ]
                  },
                  {
                    "featureType": "poi.park",
                    "elementType": "labels.text.fill",
                    "stylers": [
                      {
                        "color": "#616161"
                      }
                    ]
                  },
                  {
                    "featureType": "poi.park",
                    "elementType": "labels.text.stroke",
                    "stylers": [
                      {
                        "color": "#1b1b1b"
                      }
                    ]
                  },
                  {
                    "featureType": "road",
                    "elementType": "geometry.fill",
                    "stylers": [
                      {
                        "color": "#2c2c2c"
                      }
                    ]
                  },
                  {
                    "featureType": "road",
                    "elementType": "labels.text.fill",
                    "stylers": [
                      {
                        "color": "#8a8a8a"
                      }
                    ]
                  },
                  {
                    "featureType": "road.arterial",
                    "elementType": "geometry",
                    "stylers": [
                      {
                        "color": "#373737"
                      }
                    ]
                  },
                  {
                    "featureType": "road.highway",
                    "elementType": "geometry",
                    "stylers": [
                      {
                        "color": "#3c3c3c"
                      }
                    ]
                  },
                  {
                    "featureType": "road.highway.controlled_access",
                    "elementType": "geometry",
                    "stylers": [
                      {
                        "color": "#4e4e4e"
                      }
                    ]
                  },
                  {
                    "featureType": "road.local",
                    "elementType": "labels.text.fill",
                    "stylers": [
                      {
                        "color": "#616161"
                      }
                    ]
                  },
                  {
                    "featureType": "transit",
                    "elementType": "labels.text.fill",
                    "stylers": [
                      {
                        "color": "#757575"
                      }
                    ]
                  },
                  {
                    "featureType": "water",
                    "elementType": "geometry",
                    "stylers": [
                      {
                        "color": "#000000"
                      }
                    ]
                  },
                  {
                    "featureType": "water",
                    "elementType": "labels.text.fill",
                    "stylers": [
                      {
                        "color": "#3d3d3d"
                      }
                    ]
                  }
                ]
            }
            this.map.setOptions({styles: this.mapStyle});
        }

  },

  mounted: function () {
    google.maps.event.addDomListener(window, 'load', this.initMap);
  }
};
</script>

<style scoped>
.google-map {
  width: 750px;
  height: 520px;
  margin: 0 auto;
}
</style>
