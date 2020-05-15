<<<<<<< HEAD
# kalendarnl 万年历

![案列图片](http://cdn.datafxs.top/test-img.png)

# 在线演示
[an example](http://api.datafxs.top/index.html#/)

# 使用
    <template>
      <div>
        <el-form ref="infoForm" label-width="130px" :rules="rules">
          <el-form-item label="出生时间:" prop="">
            <kalendarnl :dateArr="dateArr" @changeDay="changeDay" :longInput="longInput"></kalendarnl>
          </el-form-item>
        </el-form>
      </div>
    </template>

    <script>
    import kalendarnl from '../assembly/kalendarnl.vue'
    export default {
      name: 'HelloWorld',
      data () {
        return {
          longInput: 0,
          dateArr: []
        }
      },
      components: {
        kalendarnl
      },
      methods: {
        changeDay (getBirthdayInfo) {
          console.log(getBirthdayInfo)
        }
      }
    }
    </script>
=======
# super-chainsaw
万年历vue插件
>>>>>>> dd13ebac1310ecec27d123f0fdf392800a7f276f
