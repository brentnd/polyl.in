<template>
  <div class="container-fluid h-100" id="app">
    <div class="row h-100">
      <div class="col-md-7">
        <div class="map" id="map"></div>
      </div>
      <div class="col-md-5 sidepanel">
        <div class="form-group">
          <label>Encoded Polyline</label>
          <textarea class="form-control" v-model="encodedPolyline" placeholder="oqr~FtmzuOJxjAiN~B"></textarea>
          <a
            href="#"
            class="small"
            v-on:click="coordinates = []"
            v-show="coordinates.length > 0"
          >Clear</a>
          <div class="small" v-show="coordinates.length == 0">
            Samples:
            <template v-for="(example, index) in examplePolylines">
              <a
                href="#"
                v-bind:key="index"
                v-on:click="encodedPolyline = example.polyline"
              >{{ example.name }}</a>&nbsp;
            </template>
          </div>
        </div>
        <div class="row">
          <div class="col">
            <div class="form-group" v-show="coordinates.length > 2">
              <label>Distance</label>
              <div class="input-group">
                <div class="form-control">{{ computeDistance() }}</div>
                <div class="input-group-append">
                  <div class="input-group-text">{{ units.short }}</div>
                </div>
              </div>
            </div>
          </div>
          <div class="col">
            <div class="form-group" v-show="coordinates.length > 2">
              <label>Points #</label>
              <div class="form-control">{{ coordinates.length }}</div>
            </div>
          </div>
        </div>
        <div class="form-group">
          <label>Coordinates</label>
          <table class="table">
            <thead>
              <tr>
                <th scope="col">#</th>
                <th scope="col">Distance</th>
                <th scope="col">Latitude</th>
                <th scope="col">Longitude</th>
                <th scope="col"></th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(element, index) in coordinatesWithDistances" v-bind:key="index">
                <th scope="row">
                  {{ index + 1 }}
                </th>
                <td>{{ element.dist }} {{ units.short }}</td>
                <td>
                  <input
                    class="form-control input-lg"
                    type="number"
                    step="any"
                    v-model="element.lat"
                  >
                </td>
                <td>
                  <input
                    class="form-control input-lg"
                    type="number"
                    step="any"
                    v-model="element.lng"
                  >
                </td>
                <td>
                  <button type="button" class="close" v-on:click="removeCoordinate(index)">
                    <span aria-hidden="true">&times;</span>
                  </button>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import L from "leaflet";
import "bootstrap/dist/css/bootstrap.css";
import "bootstrap-vue/dist/bootstrap-vue.css";
import "leaflet/dist/leaflet.css";
var polyline = require("@mapbox/polyline");
import * as turf from "@turf/turf";

export default {
  name: "app",
  data: function() {
    return {
      map: null,
      tileLayer: null,
      encodedPolyline: "",
      examplePolylines: [
        { name: "Chicago", polyline: "oqr~FtmzuOJxjAiN~B" },
        {
          name: "Long Ride",
          polyline:
            "up|sF`v|kMW{B_B^W{BwDz@KBM@gD\\eBPaBPgCVK?M@{AP_BRMBO@oAJ{@Dq@BiL@Q@O?M?oBDYBU@MB@N@FPhCj@zHLbCXvDRdDLrBFhK?T?NCLEL{@tBSb@Yh@KLIHq@ZMJKLCFEHGRGXEZATAREzAAx@@rAAp@IxAAd@Cv@@h@@h@HbBLdC?XAf@A\\Ez@C`@KhAMxBCl@_AxOs@bMEv@c@zHEt@KbDCv@ANIhCAVCTCZEZ]jC_@tC_@bDCP?P?\\?f@AT@JBXDZFXFXJ\\h@fBRp@HZF\\FXLfABNzAdIHd@Lx@BT@R@PB~@@TBRDPFNDJINq@tBk@nBM^k@hCg@~BIj@Gj@Ch@AR?p@BXBR@HF^HZrAdDL`@Hb@Fb@RfDD|B@v@AX?NCz@@f@B`@Fd@?^?`@Cb@G^i@nCwAnHcAdFQj@M^a@~@k@jA{@hBuAxCUd@R^j@z@R\\?D@D?J_@tD?N@HBPDFDFhA|@b@b@TV\\\\h@f@dCrBp@h@n@`@`@TbCbAn@ZJDTJeAzBqBpEKTw@|Au@~AMZSZUTYRUTc@ZMPa@f@[^_@f@MNi@~@a@v@OTcAhBi@`AYh@KV[p@Wl@?BOb@iArCoApDCHsAhDmBjFy@rBo@bBeApCc@fAa@hAa@`AqAxCs@|AcAnB{BlEaBbDQVQPWPKFa@JqBb@QFSLOLORwBdEIJMHMBMBWCWGWMSMUTu@d@_@XCDEJEPAR?HFtAAXCXGVU|@]|@k@zAMTQTQPUN_ChA_Bn@]REBIHINGPCRGz@M`BK~@i@~C]fCIbAATMnCA^UpECPCHTv@^dAIZc@jAm@hBw@fDKh@EX?DAV@z@?rCCn@ShCMpB?V@N?B@DDXPf@Pb@Vv@FRBR?Z?TAjBCvAAj@A`@?REXETKXqAfByAnBOPeB`C}@rAQ`@M^U~@ETOt@W~BW~AsCfPSvABp@?v@AHC\\AHAJIXO\\KXs@fBWn@_BzDaA|B[p@Ud@`DjDDDnAvAd@p@Zd@`@p@JLTV\\V^Tb@Rd@NfATbCh@VH\\JTLTPl@l@Z^rArB`@n@Xd@JLVV^R{@`FfBfAfBlApBlAF@B?HADGb@s@HGJEJAJ?DBd@TrAv@h@\\fBbAb@ZtBlBZX`BtAZV`Av@f@`@dEnDzClCtAjAjDxCHLFP@F@LDvCBTHZDFJHrIdGlClBRRVTRXl@bAf@f@ZX\\VpA|@DBf@Vp@Vb@R^R^XjDrCfDjClGdFbB~AdBzAxA~AHJdAtAX^h@YTEXKd@[\\Y\\]tAgBHEJCH?HBt@|@NNRJRJRBP@RCPENIr@k@NO\\[FIBCpAiAlAmAZ]t@w@??NWFR|@zBn@zAj@bBn@~BBLb@nABFT`@hArBpB~Cx@jAtEdFjBlBTRt@x@hFvFtBrBx@t@pBjBzA~ArBtCrA`BR^BDP\\f@rAFXRlA@r@EXWr@i@~ACJAN?TBTZv@n@jAZd@\\XhAdAVVBBfDrBl@f@fA~@VVTXrAbCtA`Ch@~@Tb@W^S^Yh@Wh@Sl@Ut@Qr@Gb@a@hE[fDKh@GXM^M^U`@SXc@h@g@d@SL`BzC|AdCHRDPBT?TG|A?l@@f@BRHjAJbB?jC@dBAr@OzCE`@Kd@AR@J?FBRDNrApC`ClEfCzEpDhGpAdCtA~BlBdDl@lA`@p@d@z@DHHHJFLDF@JALCb@WjAm@~ByAxBoAhCcBHCNAH?JBJD\\`@Z`@d@j@LLLJL?VAZKdBq@^M`@MTGl@QXIB?^GI^If@uAxHM\\c@|AkAxDqD|LOh@Rb@`@bAr@`BzAxDjBvEjApCz@~Bz@xBdBdEp@~Ad@fAhArCh@rApBpFrDnKnA`Ed@dBd@tAd@dBTv@JVNn@~@zDThAdApEPv@RrAFZzFt^Nv@BHDPFJi@l@[b@u@hAaA~AOPQLOFQBQ@uAE]BYFYHOFEBQLKHi@d@i@l@INITGVEXAR?J@`@v@hAhAtB`AhBfBlD`@t@l@lAT\\XZ`BrB`@^f@TvAt@d@XTRj@l@~@jA|AvA\\^Xb@Zf@Pd@XjAFl@Dp@@RAZCd@ARE\\A\\A^@\\D\\XjBRfAFj@b@rA@FRV^`@vInJtAxArAvAz@bAhBnBpAxAfEtEj@f@d@j@n@t@PRvDbE|@`Ax@~@PVNXlA~CbAvCR`@b@t@PRd@j@l@t@r@z@hArAb@f@JJHJJRT\\Rf@Zp@x@fB`BjDNf@F`@DRj@pCPp@`AbEJZHVDNZh@Zt@VbAl@tBvAvEL`@h@|Af@xAn@nBr@fCzAtFhAnEbChKzBfH`ChHr@~BhA`EXdAf@zAN`@Tb@RZj@r@`A`A|@fALRR\\n@vAb@fAXl@`@f@d@b@b@f@p@fAl@dArAbDL\\d@h@TRr@^|@l@OpAAtAJzC?bACx@Gn@Mn@q@rCGl@Gh@AxAF|@NjADPDRPl@\\r@t@jAh@z@\\d@pB|BnAtAf@p@Zf@tAxCd@~@n@hAZn@Rd@Pl@R|@t@xDDLLh@Fp@Hx@DlA@x@?f@Ch@I|@OdAQj@Yp@_@d@UROJ]LSFSB{Df@}AN{AH_BHs@R]NOJk@h@INGNERCP?L?Ld@xG?Z@p@?TAF?HALIh@If@m@rB[|@a@|@K\\GZG^Cb@Af@?NA\\Bf@B`@h@nE\\zBLbAHdAFtH?H@VBPDPFRHLLJFDb@^d@b@ZVrBfCLLRHJFfCpAfDfBZLZJRBn@Dl@@j@AhBKnBQHh@Dj@@b@B`@Hn@FVHZD\\@V?@Al@Mt@Oh@y@vBQ`@a@hAKd@E^?f@Ex@OxBEr@@rBATCLITKTYh@w@fBWRw@XuAf@aA^SJEBIFEHCJAH@JBPV|@@HAJCLGJE@s@NcDf@aG`Ai@Jw@V]LSJMPKRA@Wb@IVCX?rDATCXMd@mAtEU~@g@xBi@rDIt@A^?\\Dv@^vC@PBZANAPiBbOm@tFUvAQr@M`@Sd@_ApBoB~D_BlDw@bBaC`FoDvHkIhQsD|HG^CJARQlMBt@DlAb@bFV~C?Z?\\C`@o@~Gi@|FCd@?f@Bd@RdDDb@NjBNvB@^A^M~@WbBw@vFEPKTKVKPQd@QZUf@U^]j@a@~@WzASvAo@jHA`@?`@@h@LrD\\nIZbKZvK?PwGpBmCz@Dx@ZlJJrD~@dYf@`PVrLFtE@V?l@Dz@xCDp@AbAAhAGb@A|CMzFQzAE|AGZ?X?VDv@Pb@Jr@TBBPHb@\\b@\\\\P\\N|Ah@vAj@bGjCLFVDPBr@CdBKb@EGjDANC~BEhBGfAUfDUpC]`EEl@b@DdC^D@D?rBXvC\\|@Ln@Jd@@TCl@Gr@Qz@O|@OlBWn@K`AEl@AL?H@NDPHPRZ\\LLZZR`AL^b@hAFR`@pA^bBPz@LjAD^\\~EPvCN|AFn@Lz@Hb@J`@DJJTZn@xAhCbAfBLXVj@L\\HVn@nCZtA`@dBH^ThAF^L`AFl@TtCFb@DVFPVj@t@xAb@|@n@`A`@^\\Tb@TNFj@JnAL^Dr@JRLTR^^j@x@xBlCx@xA`@t@rA|BfAnBjAvBn@jAv@hBTd@cCfC}EvFmLtMaAfAuB|BaAhA{CdDyLnMeCpCmFxFg@h@GFoApAaBjBwAxAy@v@qBxBq@z@EDq@z@}BbCo@p@c@h@uC`D_AhAaChCY^~@~ARb@HXFZ@V?`@A`@IpBU`GWzFe@|La@|JUfFYvGKvDUtEq@lSA`@CpACZARaBtU_B~TUnDaAjNQhCU~CE`@MvBC~@CvB?xABxAf@bTB|@{AFoBJ{CRI?c@Cd@fMJrCHpCNjDRnFFpBB^Bx@R~FFnAFhBj@fN@f@PxFLxCxAx`@TjGj@~Ol@tOPxET`HNfEDx@Fx@@J@JDZN~@ZhAJXPn@JXt@|AhAbC`BpDh@fAXj@p@vApCzFfHdOJVHZHb@BVDx@?dAHdF?X@TDd@BNBJN`@NZdAnBpBxCn@`ARd@JXNr@DRBTDn@@fCAp@Cr@Al@?v@B|@b@jNBj@BPLj@pBlIjA`FRbAd@U^O|AdIh@tCn@lDvE~UhAdGt@|DP|@^vBF^@^@VAV?\\Cd@Ev@Ap@@V@V@BBNHXNXXRhA\\VJ^JZFjARF?n@JbAPbANh@JNFVLRNRNVVfApA^f@HRFTBLDZD|@JfDFrCBp@DTBFDHFFJDLDR@J?`@Bz@@lAA\\@F@DBDBFFDLBD@LBb@DdCFtBHnBH|BNnEJ~CXjIJnCNzFNtDN`DH`B?DHjBRvD@p@HxDHtDDrC@~@?\\J`M@`A?n@{@DgBDwDJe@?SEOCAP@\\\\jCJr@XfBHd@NzAJjAJlAJ|@F^DXJh@HXPl@rA`D~ChIhAvCp@bBTp@n@|Ab@lA`@dAN`@Zv@Tf@FLHHHHHDZFnARBb@TbHBd@Dd@RtAh@hDNfAHb@Nl@h@`BZ~@N`@FLhAzBFNLZH\\h@xCd@fDRpA?FTxAj@dEt@tE`@rCVbB|@rHl@xENjA@Px@bFFb@NbANxAL`ANtAh@|DF`@zA_@d@KL?H?JDHDDFFJFLFPNx@DXBRO`@y@rB{AbDiBjDeAjBeC`Ec@bAs@`BsDnKENCFCHk@nB[dBM~@E`AC~AExBNbJO|Dq@lPIlCC`BUtJCr@OnCKbAm@`EOdA_@xB}@rEaBxHW|@u@dCuArEiA|Dm@fCo@hCMx@U~Bu@zGCPEXAHG`@u@dCGT_GtRg@~AYfAMh@Yx@{AvC{BdEaF`Jq@jA[r@WjAMz@Mv@]bBu@rD]hAqBdGGNs@nB]x@e@`A[n@i@z@oApBcA`B}@vAiAdBSVGNEHCJABAJAb@Ad@[jKI|BALQpAg@nCKb@G\\Kf@GXYhA_@z@c@x@y@jAeBxByItKe@n@g@j@[\\j@lPJ|CHvB?DFjBDvAF~AFxBHrCFbBX|IRhFFvBVxHVbIBj@RClBK|D[NDDDDJBLVfGHnBFnBRzEZjI^nJBl@WLq@TgE~AgFtBi@TSFSHeA^uAh@EBmDrAqChAsBx@cAf@SLA@]ZLZBH@DB\\`@xKp@pRLlDFjCRhFTdHDtA?NBl@x@jVBRD^BJ@NFNdAxDx@lCJd@JZL\\h@~@\\h@V^dAzA`AtADFHPFX?LCzAClCAnFApC?nC?fAFlD@^F~@BXFZv@vD~AjHjBvIHj@jAGxDWJ?N@HFDDDLHZPfAzBzNx@bFxCnRd@~CLj@N~@^jCd@xCh@dDFl@?P?\\Cd@?TAT?R?ZBd@DXZvBhApIbBrM`@bDDb@Hl@FzABl@@\\AV?LATWzFWzFC`@BlA?f@XvLh@zPRtHJtFLpHf@f\\@f@?X@JB`@DPHRLPvBfC`BpBlBdCAl@ItBc@bLi@nNAd@Gf@l@TtBx@jBt@bCbAh@XxAx@h@^DB`Ar@bEzCdAt@\\TnAz@p@b@n@Z|@^~@\\nBl@nBr@p@Z^R`@Rx@l@x@n@|@v@zA|Aj@l@n@r@ZZr@UnF_B|DgAfDaAn@UvAc@jIeC`@ObDcAlBk@xBs@vDmAh@OnBo@pBo@`KaDdD_A|Bm@bBc@tD_A|GaBxBm@HAb@KCbJAh@AzBOnREfDU?oCZMBE@SPKJEFELCJATAhAAdA"
        }
      ],
      coordinates: [],
      polyline: null,
      units: {
        short: "m",
        full: "meters"
      }
    };
  },
  computed: {
    coordinatesWithDistances: function() {
      let distAcc = 0.0;
      let lastCoords = this.coordinates[0];
      let self = this;
      return this.coordinates.map(function(coords) {
        distAcc += turf.distance(lastCoords, coords, {
          units: self.units.full
        });
        lastCoords = coords;
        return { lat: coords[0], lng: coords[1], dist: distAcc.toFixed(1) };
      });
    }
  },
  watch: {
    encodedPolyline: function(val) {
      this.coordinates = polyline.decode(val);
    },
    coordinates: function(val) {
      if (val.length > 0) {
        if (this.polyline == null) {
          this.polyline = L.polyline(val).addTo(this.map);
        } else {
          this.polyline.setLatLngs(val);
        }
        this.map.fitBounds(this.polyline.getBounds());
        this.encodedPolyline = polyline.encode(this.coordinates);
      } else {
        this.defaultMap();
        if (this.polyline != null) {
          this.map.removeLayer(this.polyline);
          this.polyline = null;
        }
        this.encodedPolyline = "";
      }
    }
  },
  mounted() {
    this.initMap();
  },
  methods: {
    initMap() {
      this.map = L.map("map");
      this.defaultMap();
      this.tileLayer = L.tileLayer(
        "https://cartodb-basemaps-{s}.global.ssl.fastly.net/rastertiles/voyager/{z}/{x}/{y}.png",
        {
          maxZoom: 18,
          attribution:
            '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>, &copy; <a href="https://carto.com/attribution">CARTO</a>'
        }
      );
      this.tileLayer.addTo(this.map);
    },
    defaultMap() {
      this.map.setView([40.1304, -75.5149], 12);
    },
    computeDistance() {
      if (this.coordinates.length > 1) {
        var line = turf.lineString(this.coordinates);
        return turf.length(line, { units: this.units.full }).toFixed(2);
      }
    },
    removeCoordinate(index) {
      this.coordinates.splice(index, 1);
    }
  }
};
</script>

<style>
html,
body {
  height: 100%;
}
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  height: 100%;
}
.map {
  height: 100%;
}
.sidepanel {
  height: 100%;
  overflow-y: scroll;
}
</style>
