<template>
  <Form :model="formItem" :label-width="80">
    <FormItem label="视频分类">
      <Select class="tree-select">
        <v-jstree
          :data="treeData"
         
          show-checkbox
          allow-batch
          whole-row
          draggable
        ></v-jstree>
      </Select>
    </FormItem>

    <FormItem label="视频文件">
      <Row>
        <Col span="24" style="width:85px;">
          <Upload multiple action="//jsonplaceholder.typicode.com/posts/">
            <Button icon="ios-cloud-upload-outline">Upload files</Button>
          </Upload>
        </Col>
      </Row>
    </FormItem>

    <FormItem>
      <Button type="primary">提交</Button>
      <Button style="margin-left: 8px" @click="closeBtn">关闭</Button>
    </FormItem>
  </Form>
</template>
 

<script>
import Editor from "_c/editor";
import { mapMutations } from "vuex";
import { log } from "util";
import UploadVideo from "@/components/video/upload-video.vue";
import { getCourseModel, getCourseTypes } from "@/api/course";
import { all } from "q";
import VJstree from "@/components/tree/tree.vue";

export default {
  components: {
    Editor,
    VJstree,
    UploadVideo
  },
  computed: {},
  created: function() {
    // alert(this.$route.params.id)
    this.getCourseTypes();
    if (this.$route.params.id) {
      this.getCourseData(this.$route.params.id);
    }
  },
  data() {
    return {
      // tabView: "UploadVideo",
      formItem: {
        VideoName: "",
        Description: "",
        CourseType: "",
        CreateTime: "",
        editorContent: "",
        CourseStatus: 1,
        Duration: "",
        videoTableData: []
      },
      videoTableCol: [
        {
          type: "selection",
          width: 50,
          align: "center",
          key: "VideoId"
        },

        {
          title: "视频名称",
          key: "Title"
        },

        {
          title: "状态",
          key: "Status",
          width: 60,
          render: (h, params) => {
            let text = "";
            if (params.Status === "Uploading") {
              text = "上传中";
            } else if (params.Status === "UploadFail") {
              text = "上传失败";
            } else if (params.Status === "UploadSucc") {
              text = "上传完成";
            } else if (params.Status === "Transcoding") {
              text = "转码中";
            } else if (params.Status === "TranscodeFail") {
              text = "转码失败";
            } else if (params.Status === "Checking") {
              text = "审核中";
            } else if (params.Status === "Blocked") {
              text = "屏蔽";
            } else if (params.Status === "Normal") {
              text = "正常";
            }
            return h("span", text);
          }
        },

        {
          title: "创建时间",
          key: "CreationTime",
          width: 120
        },
        {
          title: "操作",
          width: 120,

          render: (h, params) => {
            //<Icon type="ios-videocam-outline" /><Icon type="md-videocam" />
            return h("div", [
              h(
                "Button",
                {
                  props: {
                    // type: "md-videocam",
                    size: "small"
                  },
                  style: {
                    // marginRight: "5px"
                  },
                  on: {
                    click: () => {
                      this.show(params.index);
                    }
                  }
                },
                "播放"
              ),
              h(
                "Button",
                {
                  props: {
                    // type: "md-videocam",
                    size: "small"
                  },
                  style: {
                    // marginRight: "5px"
                  },
                  on: {
                    click: () => {
                      this.show(params.index);
                    }
                  }
                },
                "删除"
              )
            ]);
          }
        }
      ],
      CourseTypeTreeData: [],
      CourseTypeTreeDataSelect: [],
      treeData: [
        {
          id: 0,
          text: "小学",
          value: "0",
          opened: true,
          children: [
            {
              id: 1,
              text: "语文",
              value: "initially selected",
              opened: true,
              children: [
                {
                  id: 1,
                  text: "小学语文上册",
                  value: "initially selected", opened: true,
                 children: [
                        {
                          id: 1,
                          text: "普通班",
                          value: "initially selected"
                        },

                        {
                          id: 5,
                          text: "提高班",
                          value: "custom icon"
                        },
                         {
                          id: 5,
                          text: "冲刺班",
                          value: "custom icon"
                        }
                      ]
                },

                {
                  id: 5,
                  text: "小学语文下册",
                  value: "custom icon"
                }
              ]
            },

            {
              id: 5,
              text: "数学",
              value: "custom icon"
            },
            {
              id: 5,
              text: "英语",
              value: "custom icon"
            }
          ]
        },
        {
          id: 6,
          text: "初中",
          value: "6",
          opened: true,
          children: [
            {
              id: 1,
              text: "语文",
              value: "initially selected"
            },

            {
              id: 5,
              text: "数学",
              value: "custom icon"
            },
            {
              id: 5,
              text: "英语",
              value: "custom icon"
            },
            {
              id: 5,
              text: "物理",
              value: "custom icon"
            }
          ]
        }
      ]
    };
  },
  methods: {
    ...mapMutations(["closeTag"]),
    closeBtn() {
      this.closeTag({
        name: "addcourse",
        params: {
          id: this.$route.params.id,
          title: this.$route.params.title
        }
      });
    },

    getVideoData(data) {
      let _this = this;
      data.map(function(k) {
        console.log(k);
        _this.formItem.videoTableData.push(k);
      });
    },
    //获取课程信息
    getCourseData(id) {
      this.loading = true;
      //  let p = this.courseDataTable.searchData;
      getCourseModel(id)
        .then(res => {
          let _this = this;
          // console.log(res.data.data);
          if (res.data.code != "1") {
            alert(res.data.msg);
            location.href = "/login";
          } else {
            _this.loading = false;

            _this.formItem = res.data.data.courseInfo;
            //console.log( _this.formItem);

            //_this.formItem.VideoName= res.data.data.VideoName
          }
        })
        .catch(err => {
          this.loading = false;
        });
    },
    getCourseTypes() {
      getCourseTypes()
        .then(res => {
          this.CourseTypeTreeData = [];
          res.data.data.forEach(value => {
            this.CourseTypeTreeData.push({
              id: value.ID,
              title: value.TypeName
            });
          });
        })
        .catch(err => {});
    }
  }
};
</script>
<style>
.ivu-form-item {
  margin-bottom: 10px;
}
.ivu-upload-list-file {
  border: 1px solid #ccc;
  width: 321px;

  margin-top: 10px;
}
</style>
