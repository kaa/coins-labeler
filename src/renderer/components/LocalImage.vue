<template>
  <img :src="url"/>
</template>

<script>
  import fs from 'fs'
  import path from 'path'
  import mime from 'mime'
  export default {
    props: [ "src" ],
    computed: {
      url() {
        if(this.src==null) return "";
        var mimeType = mime.getType(path.extname(this.src));
        try {
          var buffer = fs.readFileSync(this.src);
          return "data:"+mimeType+";base64,"+buffer.toString("base64");
        } catch(err) {
          return "";
        }
      }
    }
  }
</script>
