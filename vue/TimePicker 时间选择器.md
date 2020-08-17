## 前言
本文为实现 TimePicker 时间选择器 只显示时钟和分钟并做时间验证

## 例子
```html
<el-form-item :prop="'schedulingDetailList.'+index+'.startTime'" :rules="{
                 required: true, message: '起始时间不能为空', trigger: 'blur'}
                ">
                  <el-time-picker
                    v-model="schedulingDetail.startTime"
                    placeholder="起始时间"
                    size="small"
                    style="width: 100%"
                    class="date-box"
                    format='HH:mm'
                    value-format="HH:mm"
                    :picker-options="{
                    selectableRange:`00:00:00 -${schedulingDetail.endTime ? schedulingDetail.endTime+':00' : '23:59:00'}`
                    }">
                  </el-time-picker>
                </el-form-item>
              </el-col>
              <el-col :span="9">
                <el-form-item :prop="'schedulingDetailList.'+index+'.endTime'" :rules="{
                 required: true, message: '结束时间不能为空', trigger: 'blur'}
                ">
                  <el-time-picker
                    v-model="schedulingDetail.endTime"
                    placeholder="结束时间"
                    size="small"
                    style="width: 100%"
                    format='HH:mm'
                    value-format="HH:mm"
                    :picker-options="{
                    selectableRange:`${schedulingDetail.startTime ? schedulingDetail.startTime+':00' : '00:00:00'}-23:59:00`
                    }">
                  </el-time-picker>
                </el-form-item>
              </el-col>
              <el-col :span="4">
                <el-button circle icon="el-icon-plus" size="small"  @click="add()" ></el-button>
                <el-button circle icon="el-icon-minus" size="small" style="margin-left: 5%" @click="less(index)" ></el-button>
              </el-col>
            </el-row>
          </el-form-item>

          <el-form-item v-else  style="margin-top:-5%">
            <el-row :gutter="10">
              <el-col :span="9">
                <el-form-item :prop="'schedulingDetailList.'+index+'.startTime'" :rules="{
                 required: true, message: '起始时间不能为空', trigger: 'blur'}
                ">
                  <el-time-picker
                    v-model="schedulingDetail.startTime"
                    placeholder="起始时间"
                    size="small"
                    style="width: 100%"
                    class="date-box"
                    format='HH:mm'
                    value-format="HH:mm"
                    :picker-options="{
                    selectableRange:`${dataForm.schedulingDetailList[index-1].endTime ? dataForm.schedulingDetailList[index-1].endTime+':00' : '00:00:00'}
                    -${schedulingDetail.endTime ? schedulingDetail.endTime+':00' : '23:59:00'}`
                    }">
                  </el-time-picker>
                </el-form-item>
              </el-col>
              <el-col :span="9">
                <el-form-item :prop="'schedulingDetailList.'+index+'.endTime'" :rules="{
                 required: true, message: '结束时间不能为空', trigger: 'blur'}
                ">
                  <el-time-picker
                    v-model="schedulingDetail.endTime"
                    placeholder="结束时间"
                    size="small"
                    style="width: 100%"
                    format='HH:mm'
                    value-format="HH:mm"
                    :picker-options="{
                    selectableRange:`${schedulingDetail.startTime ? schedulingDetail.startTime+':00' : dataForm.schedulingDetailList[index-1].endTime ? dataForm.schedulingDetailList[index-1].endTime+':00' : '00:00:00'}-23:59:00`
                    }">
                  </el-time-picker>
                </el-form-item>
              </el-col>
              <el-col :span="4">
                <el-button circle icon="el-icon-plus" size="small" style="margin-left: 3%" @click="add()" ></el-button>
                <el-button circle icon="el-icon-minus" size="small" style="margin-left: 3%" @click="less(index)" ></el-button>
              </el-col>
            </el-row>
          </el-form-item>
        </el-form-item>
```
```javascript
dataForm: {
          schedulingDetailList: [
            {
              schedulingId: null,
              endTime: null,
              startTime: null,
            }
          ]
         }
```
## 校验
1. 开始时间不能小于上一个结束时间，不能大于结束时间
2. 结束时间不能小于开始时间或上一个结束时间，不能大于下一个开始时间
3. 非空校验

## 注意
1. 只记录时钟和分钟的话在选则范围 selectableRange 属性这里后面要加上秒数即：+ ':00'
2. 后端接受这个时间的字段要叫上两个注解：
```java
public class schedulingDetail{

	@DateTimeFormat(pattern = "HH:mm")
	@JsonFormat(pattern = "HH:mm")
	private Date startTime;
	@DateTimeFormat(pattern = "HH:mm")
	@JsonFormat(pattern = "HH:mm")
	private Date endTime;
}
```
3. 因为mysql中的 startTime 和 endTime 存储的数据格式是时间戳timestamp，因为我们只存了时钟和分钟，所以数据库中的数据是在格林威治时间 1970-01-01 08:00:00 （mysql慢8小时）加上时钟和分钟