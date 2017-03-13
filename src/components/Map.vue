<template>
    <div class="container">
        <div id="mapContainer" v-on:click="getAqi()"></div>
        <div id="pos">
            <div>
                <label>城市：</label><input type="text" id="city" v-model=city /><br>
                <label>污染指数：</label><input type="text" id="aqicn" v-model=aqi /><br>
                <label>经度:</label><input type="text" id="lngX" name="lngX" v-model=lngX /><br>
                <label>纬度:</label><input type="text" id="latY" name="latY" v-model=latY />
            </div>
        </div>
    </div>
</template>
<script>
export default {
  data () {
    return {
      lngX:'',
      latY:'',
      city:'',
      aqi:''
    }
  },
  methods:{
    getAqi(){
        var that = this;
        var token = "f7ea107c74bc8acaa9e851bd30f1ceb5f13c76fb";
        var clickEventListener=AMap.event.addListener(mapObj,'click',function(e){
            //get x, y coordinate
            var x = e.lnglat.getLng();
            var y = e.lnglat.getLat();
            that.lngX = x;
            that.latY = y;
            AMap.service('AMap.Geocoder',function(){
                var geocoder = new AMap.Geocoder({
                    city: "010"
                });
                var lnglatXY=[x, y];
                geocoder.getAddress(lnglatXY, function(status, result) {
                    if (status === 'complete' && result.info === 'OK') {
                        //get city
                        var cityName = result.regeocode.addressComponent.province;
                        that.city = cityName;
                        ajax('https://api.waqi.info/feed/shanghai/','get',{city:cityName, token: token},true)
                            .then(function(response) {
                                //get aqi
                                that.aqi = response.data.iaqi.pm25.v;
                            }, function(error) {
                                alert("Failed!", error);
                            });
                    }else{
                        alert("Failed, cannot get city");
                    }
                });  
            })
        });
       function ajax(url,type,param,async,header) {
            return new Promise(function(resolve, reject) {
                var req = new XMLHttpRequest();
                req.onload = function() { 
                    if(req.status == 200 || req.status == 304) {
                        resolve(JSON.parse(req.response));
                    } else {
                        reject(Error(req.statusText));
                    }
                };
                req.onerror = function() {
                    reject(Error("Network Error"));
                };
                type == null || type.toUpperCase() == 'GET'?type='get':type='post';
                param = formatParams(param);
                param == null || param == ''?url:url=url+'?'+param;
                async == null || async == true?async=true:async=false;
                req.open(type,url,async);
                req.send();
            });
            function formatParams(data) {
                var _fpArr = [];
                for (var _fpName in data) {
              _fpArr.push(_fpName + "=" + data[_fpName]);
                }
                return _fpArr.join("&");
            };
        }
    }
  }
}
</script>
<style type="text/css">
    body{
        margin:0;
        height:100%;
        width:100%;
        position:absolute;
        font-size:12px;
    }
    #mapContainer{
        position: absolute;
        top:0;
        left: 0;
        right:0;
        bottom:0;
    }
    #pos{
        background-color: #fff;
        padding-left: 10px;
        padding-right: 10px;
        position:absolute;
        font-size: 12px;
        left: 10px;
        top: 30px;
        border-radius: 3px;
        line-height: 30px;
        border:1px solid #ccc;
    }
    #pos input{
        border:1px solid #ddd;
        height:23px;
        border-radius:3px;
        outline:none;
    }
    #pos label{
        min-width:100px;
        display: inline-block;
        text-align:left;
    }
    #result1{
        max-height:300px;
    }
</style>