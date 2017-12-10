<template>

        <div class="row">

            <div class="col-md-3">
                <div class="row">
                    <b-card-group deckclass="mb-2">
                        <b-card bg-variant="dark" text-variant="white" header="Tools" class="text-center">
                            <b-container>
                                <div class="row">
                                    <b-btn @click="toggleIndeterminate">Calculate area</b-btn>
                                </div>
                            </b-container>
                        </b-card>
                    </b-card-group>
                </div>
                <div class="row">
                    <b-list-group>
                        <b-list-group-item v-for="layer in layers">
                            {{ layer }}
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
                            <b-nav-item-dropdown text="Funciones" right>
                              <b-dropdown-item href="#">Funcion 1</b-dropdown-item>
                              <b-dropdown-item href="#">Funcion 2</b-dropdown-item>
                            </b-nav-item-dropdown>
                          </b-navbar-nav>
                        </b-collapse>
                      </b-navbar>
                    </div>
                
                <div>

                    <div class="google-map":id="mapName"></div>

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
      fileName: '',
      selectedArea: false,
      drawingManager: null,
      marker1: null,
      marker2: null,
      poly: null,
      geodesicPoly: null,
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
              this.poly.setPath(path);
              this.geodesicPoly.setPath(path);
              var heading = google.maps.geometry.spherical.computeHeading(path[0], path[1]);
        },

        initMap(){
            const element = document.getElementById(this.mapName)
            const options = {
              zoom: 12,
              center: {lat: 19.433941, lng: -99.133563},
              styles: this.mapStyle
            }
            
            this.map = new google.maps.Map(element, options);

            this.marker1 = new google.maps.Marker({
                map: this.map,
                draggable: true,
                position: {lat: 40.714, lng: -74.006}
            });

            this.marker2 = new google.maps.Marker({
                map: this.map,
                draggable: true,
                position: {lat: 48.857, lng: 2.352}
            });

            var bounds = new google.maps.LatLngBounds(this.marker1.getPosition(), this.marker2.getPosition());
            this.map.fitBounds(bounds);

            google.maps.event.addListener(this.marker1, 'position_changed', this.update);
            google.maps.event.addListener(this.marker2, 'position_changed', this.update);
            
            this.poly = new google.maps.Polyline({
                strokeColor: '#FF0000',
                strokeOpacity: 1.0,
                strokeWeight: 3,
                map: this.map,
            });

            this.geodesicPoly = new google.maps.Polyline({
                strokeColor: '#CC0099',
                strokeOpacity: 1.0,
                strokeWeight: 3,
                geodesic: true,
                map: this.map
            });

            this.update();
        },

        updateSize(){
            this.map.data.setStyle({
              strokeWeight: this.pointSize
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
                this.layers.push(dir.split("/").slice(-1)[0]);
            }
        },

        toggleIndeterminate () {
            this.selectedArea = !this.selectedArea;
        },
        
        change_style(s){
            if(s === "standard"){
                        
            }
            else if(s === "silver"){

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

            }
            else if(s === "augbergine"){

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
    this.initMap();
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
