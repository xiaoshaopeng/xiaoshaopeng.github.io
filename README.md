# 日常笔记

## 关于表单校验正则

```js
// 常规校验
{ required: true, message: '请输入姓名', trigger: 'blur' }

// 只能数字
{ pattern: /^[0-9]*$/, message: '只能输入数字' }

// 保留小数后2位
{ pattern: /^\d+(\.\d{1,2})?$/, message:'只能保留2位', trigger: 'blur'},

// 身份证 18位
{ pattern: /^([1-9]\d{5})([1-2]\d{3})(0[1-9]|1[0-2])(0[1-9]|[1-2]\d|3[0-1])(\d{2})(\d)([0-9xX])$/, message: '请输入合法身份证' }

// 身份证 15位
{ pattern: /^[1-9]\d{5}\d{2}((0[1-9])|(10|11|12))(([0-2][1-9])|10|20|30|31)\d{3}$/, message: '请输入合法身份证' }

// 手机号
 { pattern: /^[12]\d{10}$/, message: '请输入正确的手机号', trigger: 'blur' }

// 邮箱
{ pattern: /^[0-9a-zA-Z_.-]+[@][0-9a-zA-Z_.-]+([.][a-zA-Z]+){1,2}$/, message: '请输入邮箱', trigger: 'blur' }

// 只能输入数字和字母
{ pattern: /^(\d|[a-zA-Z])+$/, message: '只能输入数字和字母' }

// 自定义规则
 {
  validator: (rule, value, cb) => {
    if (this.selfObj.bfyCertTypeCode === '1') {
      const val = checkIdNumberValid(value)
      if (val) cb()
      else cb(new Error('身份证输入有误！'))
    }
     cb()
  },
  trigger: 'blur'
}

```

## 常用日期禁用

### vue+element 版

```html
<el-date-picker
  v-model="policyEffectDateBegin"
  class="time-start"
  :picker-options="RiskStartPickerOptions"
  type="date"
  value-format="yyyy-MM-dd"
  placeholder="请选择开始时间"
/>
至
<el-date-picker
  v-model="policyEffectDateEnd"
  class="time-end"
  :picker-options="riskEndPickerOptions"
  type="date"
  value-format="yyyy-MM-dd"
  placeholder="请选择结束时间"
/>
```

```js
// 在vue中的定义
data(){
  return {
      policyEffectDateBegin: '',
      policyEffectDateEnd: '',
       // 开始
      RiskStartPickerOptions: {
        disabledDate: time => {
          if (this.policyEffectDateEnd) {
            return time.getTime() > new Date(this.policyEffectDateEnd).getTime()
          }
        }
      },
      riskEndPickerOptions: {
        disabledDate: time => {
          if (this.policyEffectDateBegin) {
            return (
              time.getTime() <
              new Date(this.policyEffectDateBegin).getTime() - 8.64e7 // 加- 8.64e7则表示包当天,去掉则可以选择当天
            )
          } else {
            return time.getTime() < Date.now()
          }
        }
      },
  }
},

```

### react-antd 版

```js
  const [startDate, setStartDate] = useState();
  const [endDate, setEndDate] = useState();

  const disabledStartDate = (curdate) => {
    console.log(curdate);
    if (!curdate || !endDate) return false;
    return curdate.isAfter(endDate, "day");
  };
  const disabledEndDate = (curdate) => {
    if (!curdate || !startDate) return false;
    return curdate.isBefore(startDate, "day");
  };

      <DatePicker
        value={startDate}
        allowClear={true}
        placeholder={""}
        onChange={(value) => {
          setStartDate(value);
        }}
        disabledDate={disabledStartDate}
      />
      <DatePicker
        value={endDate}
        allowClear={true}
        placeholder={""}
        onChange={(value) => {
          setEndDate(value);
        }}
        disabledDate={disabledEndDate}
      />
```

## 关于表格自定义 id 列

```html
<el-table-column type="index" label="ID" width="90" :index="indexMethod">
</el-table-column>
```

```js
  indexMethod(index) {
    return index + this.size * (this.page - 1) + 1
  },
```

## 关于 antd 中对 Form.List 的介绍

```jsx
import react, { useEffect } from "react";
import { Form, Space, Select, Input, Button } from "antd";
import { MinusCircleOutlined, PlusOutlined } from "@ant-design/icons";

const { Option } = Select;
const FormList = () => {
  const [form] = Form.useForm();
  useEffect(() => {
    form.setFieldsValue({
      sights: [{ sight: "Tiananmen", price: "12333" }],
    });
  }, []);
  const sights = {
    Beijing: ["Tiananmen", "Great Wall"],
    Shanghai: ["Oriental Pearl", "The Bund"],
  };
  const onFinish = () => {};

  return (
    <div>
      <Form
        form={form}
        name="dynamic_form_nest_item"
        onFinish={onFinish}
        autoComplete="off"
      >
        <Form.List name="sights">
          {(fields, { add, remove }) => (
            <>
              {fields.map((field) => (
                <Space key={field.key} align="baseline">
                  <Form.Item
                    noStyle
                    shouldUpdate={(prevValues, curValues) =>
                      prevValues.area !== curValues.area ||
                      prevValues.sights !== curValues.sights
                    }
                  >
                    {() => (
                      <Form.Item
                        {...field}
                        label="Sight"
                        name={[field.name, "sight"]}
                        rules={[
                          {
                            required: true,
                            message: "Missing sight",
                          },
                        ]}
                      >
                        <Select
                          style={{
                            width: 130,
                          }}
                        >
                          {(sights[form.getFieldValue("area")] || []).map(
                            (item) => (
                              <Option key={item} value={item}>
                                {item}
                              </Option>
                            )
                          )}
                        </Select>
                      </Form.Item>
                    )}
                  </Form.Item>
                  <Form.Item
                    {...field}
                    label="Price"
                    name={[field.name, "price"]}
                    rules={[
                      {
                        required: true,
                        message: "Missing price",
                      },
                    ]}
                  >
                    <Input />
                  </Form.Item>

                  <MinusCircleOutlined onClick={() => remove(field.name)} />
                </Space>
              ))}

              <Form.Item>
                <Button
                  type="dashed"
                  onClick={() => add()}
                  block
                  icon={<PlusOutlined />}
                >
                  Add sights
                </Button>
              </Form.Item>
            </>
          )}
        </Form.List>
      </Form>
    </div>
  );
};

export default FormList;
```

## 原生文件上传

```html
<input
  ref="fileInput"
  type="file"
  accept="application/vnd.openxmlformats-officedocument.spreadsheetml.sheet, application/vnd.ms-excel"
  @change="handleFileUpload"
/>
```

```js
handleFileUpload(e) {
  this.files = e.target.files[0]
},
```

## 下载文件

```js
const byBlobDownFile = (blob, target = false) => {
  let a = document.createElement("a");
  a.href = URL.createObjectURL(blob.data);
  target ? (a.target = "_blank") : (a.download = `错误导入账单`);
  a.click();
};
```

## 关于 antd 上传文件，不用 action，且要判断文件类型等

```jsx
<Upload
  fileList={fileList}
  beforeUpload={(file) => beforeUploadimg(file)}
  listType="picture-card"
  accept=".bmp,.gif,.png,.jpeg,.jpg,.pdf"
  onChange={handleChange}
  customRequest={uploadHeadImg}
  onRemove={(file) => {
    let arr = [...fileList];
    const index = arr.findIndex((item) => item.uid === file.uid);
    arr.splice(index, 1);
    setFileList(arr);
  }}
>
  {fileList.length >= 20 || router.query?.type === "preview" ? null : (
    <div>
      <PlusOutlined />
      <div style={{ marginTop: 8 }}>上传</div>
    </div>
  )}
</Upload>
```

```js
const beforeUploadimg: any = (file) => {
  return new Promise((resolve, reject) => {
    if (file.type === "application/pdf") {
      const pdfFileS = file.size / 1024 / 1024 < 20;
      if (!pdfFileS) {
        message.error("附件不超过20MB");
        return reject(false);
      }
    } else {
      const isLt2M = file.size / 1024 / 1024 < 3;
      if (!isLt2M) {
        message.error("每张图片不超过3MB");
        return reject(false);
      }
    }

    return resolve(true);
  });
};
```

# 关于 useEffect

## 做了什么?

组件渲染后执行的某些操作

## 为啥再组件内部调用?

可以直接访问 props、state 中的变量

## 每次都会执行么?

当 useEffect 没有传参的时候，都会更新

## effect 解决了老版 class 的业务分配逻辑

如同一个周期，mounted 可能会有不同的业务逻辑，用 useeffect 可以将里面的业务拆解出来

## 第二个参数的作用

1. 在数组中，配置上需要变动的变量，若数据变更，就会重新渲染，不变更则跳过。
2. 如果数组中有多个元素，即使只有一个元素发生变化
3. 如果传入空数组，则只执行一次

# React 父子组件的交互方式

子组件的值由父组件提供
以表单举例：将 form 传递给子组件，有 antd 自动更新。

1. 注：子组件需要在调用接口后重新渲染，需通过 state 中的变量进行改变。
2. 注：子组件若存在根据父组件的值进行变更，需在 effect 生成时进行处理

# React antd 开始结束日期控件封装

```js
import React, { useState, useEffect } from "react";
import { DatePicker } from "antd";
import { Moment } from "moment";
import "./css/index.css";

interface IDateProp {
  startDate: Moment;
  endDate: Moment;
  suffixIcon?: string;
  disabled?: boolean;
  setVal?: (type: string, par: Moment) => void;
  disabledStartDate?: () => boolean;
  disabledEndDate?: () => boolean;
  content: any;
  selfClassName?: string;
  format: string;
}

const StartEndDate: React.FC<IDateProp> = ({
  suffixIcon,
  disabled,
  startDate,
  endDate,
  disabledStartDate,
  disabledEndDate,
  setVal,
  content,
  selfClassName,
  format,
}) => {
  return (
    <div className={!selfClassName ? "date_box" : selfClassName}>
      <DatePicker
        value={startDate}
        suffixIcon={suffixIcon}
        disabled={disabled}
        allowClear={true}
        format={format}
        onChange={(val) => {
          setVal("start", val);
        }}
        disabledDate={disabledStartDate}
      />
      {content}
      <DatePicker
        value={endDate}
        suffixIcon={suffixIcon}
        disabled={disabled}
        allowClear={true}
        onChange={(val) => {
          setVal("end", val);
        }}
        format={format}
        disabledDate={disabledEndDate}
      />
    </div>
  );
};

export default StartEndDate;
```

```jsx
import StartEndDate from "./ConfigForm/modules/StartEndDate.tsx";

const [startDate, setStartDate] = useState();
const [endDate, setEndDate] = useState();
const disabledStartDate = (curdate) => {
  if (!curdate || !endDate) return false;
  return curdate.isAfter(endDate, "day");
};
const disabledEndDate = (curdate) => {
  if (!curdate || !startDate) return false;
  return curdate.isBefore(startDate, "day");
};
<StartEndDate
  startDate={startDate}
  endDate={endDate}
  setVal={(type, par) => {
    type === "start" ? setStartDate(par) : setEndDate(par);
  }}
  content={<div>至enetgi9jdfg水电费水电费</div>}
  disabledEndDate={disabledEndDate}
  disabledStartDate={disabledStartDate}
></StartEndDate>;
```

# 笔记软件

github.dev。

# 修复 element 中的选择器多选出现抖动问题

```css
/*利用sass*/
.el-select {
  position: relative;
  .el-select__tags {
    position: inherit;
    transform: translateY(0);
    padding: 3px 0;
    min-height: 28px;
  }
  .el-select__tags ~ .el-input {
    height: 100%;
    position: absolute;
    top: 50%;
    left: 0;
    transform: translateY(-50%);
    .el-input__inner {
      min-height: 20px;
      height: 100% !important;
    }
  }
  .el-select__input.is-mini {
    min-height: 20px;
  }
}
```

# 关于canvas随着适配手机变化-横屏
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
    <script src="https://cdn.staticfile.org/vue/2.2.2/vue.min.js"></script>
    <style>
      html,
      body {
        width: 100%;
        height: 100%;
        margin: 0;
        padding: 0;
      }
      #app {
        position: relative;
        width: 100%;
        height: 100%;
      }
      .sign-tip {
        width: 100vh;
        transform: rotate(90deg) translate(-100vw, -107px);
        transform-origin: 0 0;
      }
      .sign-father {
        position: relative;
        user-select: none;
        height: 100vh;
        width: 100vw;
        overflow: hidden;
      }
      .sign-canvas {
        top: 0;
        position: absolute;
        left: 73px;
      }
      .sign-page {
        transform: rotate(90deg) translateY(-100vw);
        transform-origin: 0 0;
        height: 100vw;
        width: 100vh;
      }
    </style>
  </head>
  <body>
    <div id="app">
      <div class="sign-father">
        <div class="sign-page" ref="refRotatebox">
          <div>被保人本人签字</div>
          <div class="canvasbox"></div>
          <div>以上文件必须由被保人本人签署, 请工整书写!</div>

          <div class="phone">咨询电话</div>
        </div>
      </div>
      <canvas ref="canvas" class="sign-canvas"></canvas>
      <div class="sign-tip">请在空白处签字</div>

      <button>清除</button>
      <button>确定</button>
    </div>

    <script>
      new Vue({
        el: "#app",
        data() {
          return {
            msg: "111",
          };
        },
        mounted() {
          this.init();
        },
        methods: {
          init() {
            let fabox = this.$refs.refRotatebox;
            let canvas = this.$refs.canvas;
            const width = fabox.clientHeight - 75;
            const height = fabox.clientWidth;
            canvas.width = width;
            canvas.height = height;
          },
        },
      });
    </script>
  </body>
</html>

```
