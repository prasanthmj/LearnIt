<template>
<div class="container my-4">
<div class="my-2" v-show="!flashing">
<div class="card option-card" >
  <div class="card-body">
    <h5 class="card-title">A Sample Test</h5>
    <h6 class="card-subtitle mb-2 text-muted">try a simple test</h6>
    <p class="card-text">Some Simple questions</p>
    <button class="btn btn-secondary" @click="sample()">Try this!</button>
  </div>
</div>

<div class="card option-card" >
  <div class="card-body">
    <h5 class="card-title">Upload JSON File</h5>
    
    <p class="card-text">
        Upload a JSON file with the questions and answers. 
        <a href="#">See the Format</a>.
    </p>
    <div class="file_upload_btn btn  btn-sm btn-primary" > 
    Select File
    <input type="file" name="image" @change="loadJSONFile" />
    </div>
    <div v-if="json_file_error" class="small text-danger">Error loading JSON file. The file is not formatted correctly.</div>
  </div>
</div>

<div class="card option-card" >
  <div class="card-body">
    <h5 class="card-title">Upload YAML File</h5>
    
    <p class="card-text">
        Upload a YAML file with the questions and answers. 
        <a href="#">See the Format</a>.
    </p>
    <div class="file_upload_btn btn  btn-sm btn-primary" > 
    Select File
    <input type="file" name="image" @change="loadYAMLFile" />
    </div>
    <div v-if="yaml_file_error" class="small text-danger">Error loading YAML file. The file is not formatted correctly.</div>
  </div>
</div>

</div>

<div class="mt-2" v-if="!flashing && loaded_sets.length">
<h3>Recent</h3>
<div class="card option-card" v-for="(ss,i) in loaded_sets" :key="i">
  <div class="card-body">
    <h5 class="card-title">{{ss.name}}</h5>
    
    <p class="card-text">
        {{ ss.description }}
        
    </p>
    <button class="btn btn-secondary" @click="runTest(ss)">Run Test</button>
  </div>
</div>

</div>
<flash-card :set="set" @done="close_flashing()" v-if="flashing"/>
</div>
</template>
<style lang="scss">
.option-card
{
    width:230px;
    margin:12px;
    display:inline-block;
    height:220px;
    vertical-align: top;
}
.file_upload_btn
{
    position: relative;
    overflow: hidden;
    display: inline-block;

    & >input 
    {
        position: absolute;
        opacity: 0;
        right: 0;
        top: 0; 
    }
}
</style>
<script>
import FlashCard from "../components/FlashCard";
import genq from "./q.json";
import yaml from "js-yaml";

export default {
    components:{FlashCard},
    data:()=>(
        { 
            flashing: false, 
            cur_set:{}, 
            json_file_error:false,
            yaml_file_error:false,
            loaded_sets:[]
        }),
    mounted()
    {
        this.set = genq;
        
    },
    methods:
    {
        sample()
        {
            this.set = genq;
            this.flashing=true;
        },
        runTest(set)
        {
            this.set = set;
            this.flashing=true;
        },
        close_flashing()
        {
            this.flashing=false;
        },
        loadJSONFile()
        {
            this.json_file_error=false;
            var input = event.target;
            if (!input.files || !input.files[0]) 
            {
                return;
            }
            console.log("Loaded file ", input.files[0]);
            let filename = input.files[0].name;
            var reader = new FileReader();
            reader.onload = (e) => 
            {
                let filedata = null;
                try
                {
                    filedata = JSON.parse(e.target.result);
                }
                catch(e)
                {
                    this.json_file_error=true;
                    return;
                }
                if(!filedata || !filedata.questions)
                {
                    this.json_file_error=true;
                    return;
                }
                if(!filedata.name)
                {
                    filedata.name = filename;
                }
                if(!filedata.description)
                {
                    filedata.description = "loaded from JSON File "+filename;
                }
                this.set = filedata;
                this.flashing=true;
                this.loaded_sets.push(filedata);
            }
            reader.readAsText(input.files[0]);
        },
        loadYAMLFile()
        {
            this.yaml_file_error=false;
            var input = event.target;
            if (!input.files || !input.files[0]) 
            {
                return;
            }
            console.log("Loaded file ", input.files[0]);
            let filename = input.files[0].name;
            var reader = new FileReader();
            reader.onload = (e) => 
            {
                let filedata = null;
                try
                {
                    filedata = yaml.safeLoad(e.target.result)
                    
                }
                catch(e)
                {
                    this.yaml_file_error=true;
                    return;
                }
                if(!filedata || !filedata.questions)
                {
                    this.yaml_file_error=true;
                    return;
                }
                if(!filedata.name)
                {
                    filedata.name = filename;
                }
                if(!filedata.description)
                {
                    filedata.description = "loaded from YAML File "+filename;
                }
                this.set = filedata;
                this.flashing=true;
                this.loaded_sets.push(filedata);
            }
            reader.readAsText(input.files[0]);
        }

    }
}
</script>