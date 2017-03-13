<template>
    <div class="container">
        <div id="mapContainer" v-on:click="getAqi2()"></div>
        <div id="pos">
            <div><br>
                <label>城市：</label><input type="text" id="city" v-mode=city /><br>
                <label>污染指数：</label><input type="text" id="aqinc" v-mode=aqi /><br>
                <label>经度:</label><input type="text" id="lngX" name="lngX" v-model=lngX /><br>
                <label>纬度:</label><input type="text" id="latY" name="latY" v-model=lngY />
            </div>
        </div>
    </div>
</template>
<script>
export default {
  data () {
    return {
      lngX:'',
      lngY:'',
      city:'',
      aqi:'',
      mapObj:''
    }
  },
  methods:{
      getAqi2($event){
          alert(2);
      },
    getAqi(){
        var cityName = "";
        alert(1);
        var clickEventListener=AMap.event.addListener(mapObj,'click',function(e){
            alert(e)
            this.lngX = e.lnglat.getLng();
            this.lngY = e.lnglat.getLat();
            var citysearch = new AMap.CitySearch();
            citysearch.getLocalCity(function(status, result) {
                if (status === 'complete' && result.info === 'OK') {
                    if (result && result.city && result.bounds) {
                        cityName = result.city;
                        this.city = result.city;
                        var citybounds = result.bounds;
                    }
                } else {
                    alert(result.info);
                }
            });
            ajax('https://api.waqi.info/feed/shanghai/','get',{city:cityName, token: token},true)
            .then(function(response) {
                this.aqi = response.data.iaqi.pm25.v;
            }, function(error) {
                alert("Failed!", error);
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
        });
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
        height: 170px;
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