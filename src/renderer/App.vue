<template>
  <div id="app">
    <div v-if="files">
      <div>{{ix}}</div>
      <local-image :src="files[ix]"/>
      <div>
        <button @click="ix = Math.max(0,ix-1)">Previous</button>
        <button v-for="label in labels" :key="label" @click="assign(ix,label)" :class="{ selected: isAssigned(ix,label) }">{{label}}</button>
        <button @click="ix = Math.min(ix+1, files.length-1)">Next</button>
      </div>
    </div>
    <div v-else>
      <div><input type="checkbox" v-model="includeLabeled"> Include already labeled images</div>
      <button :disabled="sourceMutex" @click="pickSource">Select source</button>
    </div>
  </div>
</template>

<script>
  import fs from 'fs'
  import glob from 'glob'
  import path from 'path'
  import { remote } from 'electron'
  const { dialog } = remote;
  import LocalImage from './components/LocalImage'

  export default {
    name: 'labler',
    data() {
      return {
        sourceMutex: false,
        sourceDir: null,
        includeLabeled: false,
        ix: 0,
        files: null,
        labels: ["none","1c","2c","5c","10c","20c","50c","1e","2e"]
      }
    },
    components: { 
      LocalImage 
    },
    methods: {
      pickSource() {
        this.sourceMutex = true;
        dialog.showOpenDialog({ properties: ['openDirectory'] }, dir => {
          this.sourceMutex = false;
          if(!dir) return;
          this.sourceDir = dir[0];
          glob((this.includeLabeled ? "**/" : "")+"*.{jpg,png}", { cwd: this.sourceDir }, (err, res) => {
            console.log(res, this.includeLabeled);
            this.files = res.map(t => path.join(this.sourceDir, t));
            this.classes = new Array(res.length);
            this.ix = 0;
          });
        })
      },
      assign(ix, label) {
        let file = this.files[ix];
        let assignedFile = path.join(this.sourceDir,label,path.basename(file));
        fs.mkdir(path.join(this.sourceDir, label), err => {
          if(err && err.code !== "EEXIST") 
            return dialog.showErrorBox("Unable to create directory for label "+label, err.message);
          fs.rename(file, assignedFile, err => {
            if(err) return dialog.showErrorBox("Unable to assign label "+label+" to image.", err.message);
            this.files[ix] = assignedFile;
            this.ix++;
          });
        })
      },
      isAssigned(ix, label) {
        var file = this.files[ix];
        console.log(ix, label, file);
        return path.dirname(path.relative(file,this.sourceDir)) === label;
      }
    }
  }
</script>

<style>
  button {
    width: 50px;
    padding: 10px 0;
    text-align: center;
  }
  button.selected {
    background: green;
  }
</style>
