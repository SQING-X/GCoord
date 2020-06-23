<template>
    <div class="gc-wrap">
        <h2>Gcoord地理坐标转换器</h2>
        <el-tag>by:Sqing</el-tag>
        <el-card class="box-card">
            <div class="tips">1. 本工具参考<el-link href='https://github.com/hujiulong/gcoord' target='_blank' type='primary'>gcoord</el-link></div>
            <div class="tips">2. 修正百度地图、高德地图及其它互联网地图坐标系不统一的问题</div>
            <div class="tips">3. 支持转换坐标数组和GeoJson数据</div>
        </el-card>
        <div class="tf-wrap">
            <div class="singletf">
                <el-card class="single-card">
                    <h2>单一坐标数组转换</h2>
                    <div>
                        <el-tag>FROM</el-tag>
                        <el-select  v-model='s_from' label='From:'>
                           <el-option v-for="(item,index) in gcZB" :value='item' :key='index'></el-option>
                        </el-select>
                    </div>
                    <div>
                        <el-tag>TO</el-tag>
                        <el-select  v-model='s_to'>
                           <el-option v-for="(item,index) in gcZB" :value='item' :key='index'></el-option>
                        </el-select>
                    </div>
                    <div>
                        <el-tag>坐标</el-tag>
                        <el-input v-model='s_input' class="el-input-data" placeHolder='经纬度，以,分隔'></el-input>
                    </div>
                    <div class="tf-button">
                        <el-button type='primary' @click="s_tf">转换</el-button>
                    </div>
                    <div>
                        <el-tag>转换结果</el-tag>
                        <el-input v-model='s_result'  readonly class="el-input-data" placeHolder='经纬度，以,分隔'></el-input>
                    </div>
                </el-card>
            </div>
           
            <div class="multitf">
                <el-card class="multi-card">
                    <h2>批量坐标数组转换</h2>
                    <div>
                        <el-tag>FROM</el-tag>
                        <el-select  v-model='m_from' label='From:'>
                            <el-option v-for="(item,index) in gcZB" :value='item' :key='index'></el-option>
                        </el-select>
                    </div>
                    <div>
                        <el-tag>TO</el-tag>
                        <el-select  v-model='m_to'>
                            <el-option v-for="(item,index) in gcZB" :value='item' :key='index'></el-option>
                        </el-select>
                    </div>
                    <div>
                        <el-tag>坐标</el-tag>
                        <el-upload
                            ref='upload'
                            accept=".xls,.xlsx"
                            action='./'
                            :on-change='readGFile'
                            :auto-upload='false'
                        >
                        <el-button size='small' type='primary'>选取文件</el-button>

                        </el-upload>
                    </div>
                    <div class="tf-button">
                        <el-button type='primary' @click="m_tf">转换</el-button>
                    </div>
                </el-card>
            </div>
        </div>
    </div>
</template>

<script>
import gcoord from 'gcoord'
import XLSX from 'xlsx'
export default {
    name:'GCView',
    data () {
        return {
            gcZB:[
                'WGS84',
                'GCJ02',
                'BD09',
                'BD09LL',
                'BD09MC',
                'BD09Meter',
                'Baidu',
                'BMap',
                'WebMercator',
                'WGS1984',
                'EPSG4326',
                'EPSG3857',
                'EPSG900913'
            ],
            file:'',
            s_from:'',
            s_to:'',
            s_input:'',
            s_result:'',
            m_from:'',
            m_to:'',
            m_input:'',
            m_result:'',

        }
    },
    methods:{
        s_tf(){
            try {
                let temp = this.s_input.split(',')
                console.log(temp)
                let data = [parseFloat(temp[0]),parseFloat(temp[1])]
                console.log(data)
                this.s_result =JSON.stringify(
                    gcoord.transform(
                        data,
                        gcoord[this.s_from],
                        gcoord[this.s_to]
                ))
            } catch (error) {
                this.$message.error('转换出错，请确认数据正确与否')
            }
        },
        readGFile(file){

             this.file = {0:file.raw}
        },
        readExcel1(files) {//表格导入
            if(files.length<=0){//如果没有文件名
                return false;
            }else if(!/\.(xls|xlsx)$/.test(files[0].name.toLowerCase())){
                this.$Message.error('上传格式不正确，请上传xls或者xlsx格式');
                return false;
            }
            const fileReader = new FileReader();
            fileReader.onload = (ev) => {
                try {
                    let resultData = []
                    const data = ev.target.result;
                    const workbook = XLSX.read(data, {
                        type: 'binary'
                    });
                    const wsname = workbook.SheetNames[0];//取第一张表
                    const ws = XLSX.utils.sheet_to_json(workbook.Sheets[wsname]);//生成json表格内容
                    console.log(ws)
                    let startIndex = ws[0].__rowNum__
                    console.log(startIndex)
                    for(let i=0;i<ws.length;i++){
                        if(ws[i].__rowNum__ == startIndex){
                            let item = [ws[i]['经度'],ws[i]['纬度']]
                            let temp = gcoord.transform(
                                item,
                                gcoord[this.m_from],
                                gcoord[this.m_to])
                            resultData.push({"经度_tf":temp[0],"纬度_tf":temp[1]})
                            startIndex++
                        }else{
                            resultData.push({"经度_tf":"","纬度_tf":""})
                            startIndex +=2
                        }
                        
                    }
                    
                    const ot = XLSX.utils.json_to_sheet(resultData)
                    const new_book = XLSX.utils.book_new()
                    XLSX.utils.book_append_sheet(new_book,ot,'已转换_'+files[0].name.split('.')[0])
                    XLSX.writeFile(new_book,'已转换_'+files[0].name.split('.')[0]+'.xlsx')
                    this.$refs.upload.value = '';


                } catch (e) {
                    this.$message.error('出错了，请检查数据')
                }
            };
            fileReader.readAsBinaryString(files[0]);
        },

        m_tf(){
            if(!this.m_from || !this.m_to){
                this.$message.error("请选择坐标系")
                return
            }
            this.readExcel1(this.file)
        }
    }
}
</script>

<style  scoped>
    .gc-wrap{
        display: flex;
        width: 100%;
        justify-content: center;
        align-items: center;
        flex-direction: column;
    }
    .box-card{
        width: 700px;
        text-align: left;
        font-size: 20px;
    }
    .box-card .tips{
        margin-top: 8px;
    }
    .tf-wrap{
        width: 89%;
        display: flex;
        justify-content: space-evenly;
        margin-top: 40px;
    }
   .tf-wrap .multitf,.singletf{
        width: 30%;
        padding-left: 10px;
        background-color: #00c2ff;
        border-radius: 5px;
    }
    .multitf{
        height: 427px;
    }
    .single-card, .multi-card{
        width: 100%;
    }
   .el-select{
       margin-right:10px;
       margin-bottom:10px;
       width:90%
   }
    .tf-wrap .el-input-data{
        width: 90%;
    }
    .el-tag{
        width: 70px;
    }
    .tf-button{
        text-align: center;
        margin-top: 40px;
    }
    .tf-button .el-button{
        width: 100px;
    }
</style>

