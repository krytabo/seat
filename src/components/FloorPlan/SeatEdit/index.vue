<template>
  <div id="app">
    <div class="home">
      <div class="fixed top-0 w-full z-50">
        <Header></Header>
        <div class="bg-white hidden md:flex flex-col md:flex-row md:ml-auto mt-3 md:mt-0 shadow-md h-12 items-center px-5" id="navbar-collapse">
          <p class="text-xl font-bold">編輯座位圖</p>

          <div class="float-right ml-auto">
            <!--            <router-link to="/about" class="mr-3">
              <el-button size="small">取消</el-button>
            </router-link>-->

            <el-button size="small" @click="Cancel_Alert">取消</el-button>

            <el-button size="small" type="primary" @click="Save_Alert">存檔</el-button>
          </div>
        </div>
      </div>
      <div class="pt-40 pb-10 px-5 md:px-11 bg-white h-screen flex justify-center">
        <!--左邊新增-->
        <div class="flex-none w-52">
          <el-descriptions class="mb-50" title="新增座位" :column="1" direction="vertical">
            <el-descriptions-item label="目前排序">
              <el-input size="mini" v-model="number"></el-input>
            </el-descriptions-item>
            <el-descriptions-item label="座位高度">
              <el-input-number v-model="h" :step="40" step-strictly></el-input-number>
            </el-descriptions-item>
            <el-descriptions-item label="座位寬度">
              <el-input-number v-model="w" :step="40" step-strictly></el-input-number>
            </el-descriptions-item>
            <el-descriptions-item label="文字顏色">
              <div class="color-picker font-color-picker">
                <el-color-picker v-model="textcolor"></el-color-picker>
              </div>
            </el-descriptions-item>
            <el-descriptions-item label="背景顏色">
              <div class="color-picker back-color-picker">
                <el-color-picker v-model="backgroundColor"></el-color-picker>
              </div>
            </el-descriptions-item>
          </el-descriptions>

          <el-button type="danger" @click.native="DeleteAll_Alert()">全部清除</el-button>
          <el-button type="primary" @click.native="addseat()">添加座位</el-button>

          <el-divider></el-divider>

          <div class="md:col-span-1">
            <div class="px-4 sm:px-0 text-left">
              <h3 class="text-base font-medium leading-6 text-gray-900">上傳參考圖</h3>
              <div class="mt-1 text-sm text-gray-600">
                <el-upload
                  action="#"
                  list-type="picture-card"
                  :auto-upload="false"
                  ref="uploadImg"
                  :on-change="toggleUpload"
                  :on-remove="toggleUpload"
                  class="overflow-hidden"
                  style="max-height: 148px"
                >
                  <i slot="default" class="el-icon-plus"></i>
                  <div slot="file" slot-scope="{ file }">
                    <img class="el-upload-list__item-thumbnail" :src="file.url" alt="" />
                    <span class="el-upload-list__item-actions">
                      <!--圖片放大-->
                      <span class="el-upload-list__item-preview" @click="imagePreview(file)">
                        <i class="ri-zoom-in-line"></i>
                      </span>
                      <!--圖片開啟-->
                      <span v-if="!disabled" class="el-upload-list__item-delete" @click="handlePictureCardPreview(file)">
                        <i class="ri-eye-line"></i>
                      </span>

                      <!--圖片關閉 -->
                      <span v-if="!disabled" class="el-upload-list__item-delete" @click="imageClose(file)">
                        <i class="ri-eye-off-line"></i>
                      </span>
                      <!--圖片刪除-->
                      <span v-if="!disabled" class="el-upload-list__item-delete" @click="handleRemove(file)">
                        <i class="ri-delete-bin-5-line"></i>
                      </span>
                    </span>
                  </div>
                </el-upload>
              </div>
            </div>
          </div>

          <el-dialog :visible.sync="dialogVisible">
            <img width="100%" :src="ImagePreviewUrl" alt="" />
          </el-dialog>
        </div>
        <!--中間座位-->
        <div class="flex-grow w-16 mx-5">
          <div class="elements drag-wrap overflow-auto h-full bg-gray-200">
            <!--            <vue-infinite-viewer class="elements infinite-viewer drag-wrap" ref="viewer">-->
            <div
              v-cloak
              style="height: 100vh; background-repeat: no-repeat; width: 100vw; object-fit: cover"
              :style="{ backgroundImage: 'linear-gradient(rgba(255, 255, 255, 0.6), rgba(255, 255, 255, 0.6)), url(' + dialogImageUrl + ')' }"
            >
              <!--<div class="bg-grid"></div>-->
              <vdr
                v-for="(item, i) in controlsArr"
                :grid="[20, 20]"
                :key="i"
                :ref="controlsArr"
                :w="item.w"
                :h="item.h"
                :parentH="600"
                :parentW="800"
                :parent="true"
                :x="item.x"
                :y="item.y"
                :class-name-active="activedClass"
                @dragstop="onDragstop"
                class="test1"
                :resizable="false"
                @dragging="onDrag"
                :is-conflict-check="true"
                :snap="true"
                @refLineParams="getRefLineParams"
                @activated="handleShowInfo(item)"
                :style="{
                  backgroundColor: item.backgroundColor,
                  color: item.textcolor,
                }"
              >
                <p style="margin-bottom: 0">{{ item.customId }}</p>

                <!-- 刪除 -->
                <i class="ri-close-circle-fill close" @click="removeControl(item.customId)"></i>
              </vdr>

              <div
                class="ref-line v-line"
                v-for="item in vLine"
                v-show="item.display"
                :key="item.customId"
                :style="{
                  left: item.position,
                  top: item.origin,
                  height: item.lineLength,
                }"
              ></div>
              <span
                class="ref-line h-line"
                v-for="item in hLine"
                v-show="item.display"
                :key="item.customId"
                :style="{
                  top: item.position,
                  left: item.origin,
                  width: item.lineLength,
                }"
              ></span>
            </div>
          </div>
          <!--            </vue-infinite-viewer>-->
        </div>
        <!--右邊編輯-->
        <div class="flex-none w-52">
          <el-descriptions class="mb-50" title="變更內容" :column="1" direction="vertical">
            <template slot="extra">
              <el-button v-if="controlsArr.x && singleState.customId > 0" type="danger" size="small" @click.native="removeControl()" plain>刪除</el-button>
              <el-button v-else type="danger" size="small" disabled plain>刪除</el-button>
            </template>

            <el-descriptions-item label="文字顏色">
              <div class="color-picker font-color-picker">
                <el-color-picker v-model="singleState.textcolor"></el-color-picker>
              </div>
            </el-descriptions-item>
            <el-descriptions-item label="背景顏色">
              <div class="color-picker back-color-picker">
                <el-color-picker v-model="singleState.backgroundColor"></el-color-picker>
              </div>
            </el-descriptions-item>
            <el-descriptions-item label="位置序號">
              <el-input size="mini" v-model="singleState.customId"></el-input>
            </el-descriptions-item>

            <!--<el-descriptions-item label="座標Ｘ">
              <el-input size="mini" v-model="singleState.y"></el-input>
            </el-descriptions-item>-->

            <el-descriptions-item label="座位高度">
              <el-input size="mini" v-model="singleState.h"></el-input>
            </el-descriptions-item>
            <el-descriptions-item label="座位寬度">
              <el-input size="mini" v-model="singleState.w"></el-input>
            </el-descriptions-item>
          </el-descriptions>

          <el-divider></el-divider>

          <el-descriptions class="mb-30" title="目前位置" :model="controlsArr" :column="2">
            <el-descriptions-item label="Ｘ">{{ controlsArr.x }}</el-descriptions-item>
            <el-descriptions-item label="Ｙ">{{ controlsArr.y }}</el-descriptions-item>
          </el-descriptions>

          <el-divider></el-divider>

          <div class="keys">
            <div class="up arr">
              <el-button v-if="singleState.customId > 0" type="danger" @click.native="removeOneByOne()" plain><i class="ri-delete-back-2-line"></i></el-button>
              <el-button v-else type="danger" disabled plain><i class="ri-delete-back-2-line"></i></el-button>
            </div>
            <div class="up arr">
              <el-button v-if="controlsArr.y && singleState.customId > 0" type="primary" @click.native="AddControlTOP()"><i class="ri-arrow-up-line"></i></el-button>
              <el-button v-else type="primary" disabled><i class="ri-arrow-up-line"></i></el-button>
            </div>
            <br />
            <div class="left arr">
              <el-button v-if="controlsArr.x && singleState.customId > 0" type="primary" @click.native="AddControlLEFT()"><i class="ri-arrow-left-line"></i></el-button>
              <el-button v-else type="primary" disabled><i class="ri-arrow-left-line"></i></el-button>
            </div>
            <div class="bottom arr">
              <el-button v-if="singleState.customId > 0" type="primary" @click.native="AddControlBOTTOM()"><i class="ri-arrow-down-line"></i></el-button>
              <el-button v-else type="primary" disabled><i class="ri-arrow-down-line"></i></el-button>
            </div>
            <div class="right arr">
              <el-button v-if="singleState.customId > 0" type="primary" @click.native="AddControlRIGHT()"><i class="ri-arrow-right-line"></i></el-button>
              <el-button v-else type="primary" disabled><i class="ri-arrow-right-line"></i></el-button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Header from "@/components/Header/index.vue";
import vdr from "vue-draggable-resizable-gorkys";
import { findIndex } from "core-js/internals/array-iteration";
import { VueInfiniteViewer } from "vue-infinite-viewer";

export default {
  name: "SeatEdit",
  components: {
    Header,
    vdr,
    VueInfiniteViewer,
  },
  data: function () {
    return {
      dialogImageUrl: "",
      ImagePreviewUrl: "",
      dialogVisible: false,
      disabled: false,

      // scrollOptions: {},

      // 輔助線
      vLine: [],
      hLine: [],

      // colorPicker外觀改變
      color: "#181515",

      // 添加座位使用
      activedClass: "lq-active-class",
      controlsArr: [], // 所有控件数组
      singleState: {}, // after click get data
      number: 1,
      Profiler: false,
      x: "",
      y: "",
      customId: this.number,
      w: 40,
      h: 40,
      type: "",
      money: "",
      textVal: "",
      backgroundColor: "#eee",
      textcolor: "#333",
    };
  },
  created: function () {
    this.$store.state.leftbackground = "#eee"; //默認背景顏色
    this.$store.state.leftcolor = "#333"; //默認文字顏色
  },
  watch: {
    controlsArr: {
      handler() {
        this.onComplete();
      },
      immediate: true,
      deep: true,
    },
  },
  methods: {
    // =======Sweetalert=======

    // swal-取消
    Cancel_Alert() {
      this.$swal
        .fire({
          title: "你確定嗎？",
          text: "檔案尚未存檔，資料將會清空！",
          icon: "warning",
          showCancelButton: true,
          cancelButtonColor: "#0084ff",
          confirmButtonColor: "#cccccc",
          confirmButtonText: "確定離開",
          cancelButtonText: "繼續填寫",
          closeOnConfirm: false,
        })
        .then((result) => {
          if (result.isConfirmed) {
            // this._router.navigate(["/about"]);
            window.location.href = "#/about";
          }
        });
    },

    // 存檔
    Save_Alert() {
      this.$swal
        .fire({
          title: "存檔完成！",
          icon: "success",
          showCancelButton: true,
          confirmButtonColor: "#0084ff",
          confirmButtonText: "返回列表",
          cancelButtonText: "繼續編輯",
        })
        .then((result) => {
          if (result.isConfirmed) {
            // this._router.navigate(["/about"]);
            window.location.href = "#/about";
          }
        });
    },

    // swal-刪除
    Delete_Alert() {
      // Use sweetalert2
      this.$swal
        .fire({
          title: "你確定嗎？",
          text: "資料刪除後將無法復原",
          icon: "warning",
          showCancelButton: true,
          cancelButtonColor: "#0084ff",
          confirmButtonColor: "#cccccc",
          confirmButtonText: "確定刪除",
          cancelButtonText: "取消",
          closeOnConfirm: false,
        })
        .then((result) => {
          if (result.isConfirmed) {
            this.$swal.fire("刪除成功!", "您的檔案已成功刪除", "離開");
          }
        });
    },

    // swal-一鍵清除
    DeleteAll_Alert() {
      this.$swal
        .fire({
          title: "你確定嗎？",
          text: "所有座位將會清空！",
          icon: "warning",
          showCancelButton: true,
          cancelButtonColor: "#0084ff",
          confirmButtonColor: "#cccccc",
          confirmButtonText: "確定清除",
          cancelButtonText: "取消",
          closeOnConfirm: false,
        })
        .then((result) => {
          if (result.isConfirmed) {
            this.removeAll();
          }
        });
    },

    // =======圖片相關=======

    // 圖片背景關閉
    imageClose(file) {
      console.log(file);
      this.dialogImageUrl = "";
    },

    // 圖片背景開啟
    handlePictureCardPreview(file) {
      this.dialogImageUrl = file.url;
    },
    // 圖片刪除
    handleRemove(file) {
      console.log(file);
      this.$refs.uploadImg.clearFiles(); //移除已上傳圖片
    },
    // 圖片放大預覽
    imagePreview(file) {
      this.ImagePreviewUrl = file.url;
      this.dialogVisible = true;
    },

    toggleUpload() {
      this.showUpload = !this.showUpload;
    },

    // 添加座位
    /* addControl() {
        console.log();
        this.controlsArr.push({
          customId: this.number++,
          textVal: "",
          y: this.onDragstop(),
          x: this.onDragstop(),
        });
      }, */

    addseat: function () {
      this.controlsArr.push({
        customId: this.number++,
        textVal: this.textVal,
        y: 0,
        x: 0,
        type: this.type,
        money: this.money,
        w: this.w,
        h: this.h,
        backgroundColor: this.backgroundColor,
        textcolor: this.textcolor,
      });
    },

    AddControlTOP(x, y) {
      x = this.controlsArr.x;
      y = this.controlsArr.y;

      this.controlsArr.x = x + 0;
      this.controlsArr.y = y - this.h;
      this.controlsArr.push({
        customId: this.number++,
        textVal: this.textVal,
        y: this.controlsArr.y,
        x: this.controlsArr.x,
        type: this.type,
        money: this.money,
        w: this.w,
        h: this.h,
        backgroundColor: this.backgroundColor,
        textcolor: this.textcolor,
      });
    },

    AddControlBOTTOM(x, y) {
      x = this.controlsArr.x;
      y = this.controlsArr.y;

      this.controlsArr.x = x + 0;
      this.controlsArr.y = y + this.h;
      this.controlsArr.push({
        customId: this.number++,
        textVal: this.textVal,
        y: this.controlsArr.y,
        x: this.controlsArr.x,
        type: this.type,
        money: this.money,
        w: this.w,
        h: this.h,
        backgroundColor: this.backgroundColor,
        textcolor: this.textcolor,
      });
    },

    AddControlLEFT(x, y) {
      x = this.controlsArr.x;
      y = this.controlsArr.y;

      this.controlsArr.x = x - this.w;
      this.controlsArr.y = y + 0;
      this.controlsArr.push({
        customId: this.number++,
        textVal: this.textVal,
        y: this.controlsArr.y,
        x: this.controlsArr.x,
        type: this.type,
        money: this.money,
        w: this.w,
        h: this.h,
        backgroundColor: this.backgroundColor,
        textcolor: this.textcolor,
      });
    },

    AddControlRIGHT(x, y) {
      x = this.controlsArr.x;
      y = this.controlsArr.y;

      this.controlsArr.x = x + this.w;
      this.controlsArr.y = y + 0;
      this.controlsArr.push({
        customId: this.number++,
        textVal: this.textVal,
        y: this.controlsArr.y,
        x: this.controlsArr.x,
        type: this.type,
        money: this.money,
        w: this.w,
        h: this.h,
        backgroundColor: this.backgroundColor,
        textcolor: this.textcolor,
      });
    },

    // 刪除已選座位
    removeControl(customId) {
      const editIndex = findIndex(this.controlsArr, (o) => {
        return o.customId === this.singleState.customId;
      });
      this.controlsArr.splice(editIndex, 1);
      this.singleState = "";
      /*let editIndex = findIndex(this.controlsArr, (o) => {
        return o.customId === customId;
      });
      this.controlsArr.splice(editIndex, 1);*/
    },

    // 全部清除
    removeAll(customId) {
      this.controlsArr.splice(0);
    },

    // 逐一刪除
    removeOneByOne(customId) {
      const editIndex = findIndex(this.controlsArr, (o) => {
        return o.customId === this.singleState.customId;
      });
      this.controlsArr.splice(editIndex, 1);
    },

    // 點擊出現資訊
    handleShowInfo(item) {
      this.singleState = item;
      console.log(item);
    },

    // 移動結束、點擊出現
    onDragstop(x, y, element, row, item) {
      this.controlsArr.x = x;
      this.controlsArr.y = y;
      this.singleState.x = x;
      this.singleState.y = y;
      this.x = x;
      this.y = y;
      this.w = row.x;

      // 起始座標變更為最後一次的位置
      /* while (element) {
          x += element.offsetLeft - element.scrollLeft + element.clientLeft;
          y += element.offsetTop - element.scrollLeft + element.clientTop;
          element = element.offsetParent;
        }
        return { x: x, y: y };*/
      //  起始座標變更結束
    },

    /*    edit: function () {
      // this.controlsArr = Object.assign({}, row);
      this.$set(this.controlsArr);
    },*/

    // 輔助線
    getRefLineParams(params) {
      const { vLine, hLine } = params;
      this.vLine = vLine;
      this.hLine = hLine;
    },

    onResize: function (x, y, width, height) {
      this.x = x;
      this.y = y;
      this.width = width;
      this.height = height;
    },
    onDrag: function (x, y) {
      this.controlsArr.x = x;
      this.controlsArr.y = y;
    },
  },
  mounted() {
    this.getDomData();
    window.addEventListener("keydown", (ev) => {
      if (ev.keyCode === 17) {
        this.sync = true;
      }
    });
    window.addEventListener("keyup", (ev) => {
      if (ev.keyCode === 17) {
        this.sync = false;
      }
    });
  },
  computed: {},
};
</script>

<style lang="scss">
.hideUpload > div {
  display: none;
}
[v-cloak] {
  display: none;
}

.keys {
  width: auto;
  height: auto;
}

.arr {
  display: inline-block;
  margin: 5px;
}

.up {
  position: relative;
  top: -4px;
  left: -33px;
}

.test1 {
  background-color: rgb(239, 154, 154);
  border-radius: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.test2 {
  background-color: rgb(233, 154, 154);
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  height: 100vh;
}

#nav {
  padding: 30px;

  a {
    font-weight: bold;
    color: #2c3e50;

    &.router-link-exact-active {
      color: #42b983;
    }
  }
}

.drag-test {
  display: flex;
  /*justify-content: space-between;*/
  position: absolute;
  left: 20px;
  top: 20px;
  right: 20px;
  bottom: 20px;

  .controls-list {
    width: 300px;
    padding: 0 10px;
    border-right: solid 1px rgb(105, 103, 103);

    .controls-item {
      height: 30px;
      border: solid 1px #ccc;
      margin-bottom: 20px;
      cursor: pointer;
      line-height: 30px;
      text-align: center;
    }
  }
}

.bg-grid {
  background: linear-gradient(-90deg, rgba(0, 0, 0, 0.1) 1px, transparent 1px) 0% 0% / 40px 40px, linear-gradient(rgba(0, 0, 0, 0.1) 1px, transparent 1px) 0% 0% / 40px 40px;
  width: 100%;
  height: 100%;
  position: absolute;
}
.drag-wrap {
  width: 100%;
  height: 100%;
  border: solid 1px #ccc;
  position: relative;
  //background: linear-gradient(-90deg, rgba(0, 0, 0, 0.1) 1px, transparent 1px) 0% 0% / 40px 40px, linear-gradient(rgba(0, 0, 0, 0.1) 1px, transparent 1px) 0% 0% / 40px 40px;
  //margin: 0 20px;

  .draggable {
    cursor: move;
  }

  .inner-container {
    margin-top: -20px;
    height: 20px;
    font-size: 14px;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
    position: relative;

    .name {
      font-size: 12px;
    }
  }

  .el-icon-error {
    font-size: 16px;
    position: absolute;
    top: -20px;
    right: -10px;
    z-index: 100;
    cursor: pointer;
  }

  .lq-draggable-text,
  .lq-draggable-date,
  .lq-draggable-sign,
  .lq-draggable-select {
    border: dashed 1px #000;
  }

  .lq-draggable-seal {
    .seal-inner {
      width: 100%;
      height: 100%;
      border: dashed 1px #000;

      .seal {
        width: calc(100% - 12px);
        height: calc(100% - 12px);
        margin: 5px 0 0 5px;
        border: dashed 1px #000;
        border-radius: 50%;
      }
    }
  }

  .lq-active-class {
    //border-color: rgb(14, 74, 238);

    .seal-inner {
      border-color: rgb(14, 74, 238);

      .seal {
        border-color: rgb(14, 74, 238);
      }
    }
  }
}

.vdr {
  border: 1px solid #fff !important;
}

.set-wrap {
  width: 230px;
  padding: 0 10px;
  border-left: solid 1px rgb(105, 103, 103);

  .control-set-item {
    margin-bottom: 20px;

    .title {
      margin-bottom: 5px;
      font-size: 14px;
    }
  }

  .mb-50 {
    margin-bottom: 50px;
  }
}

.lq-dragging-class {
  border: dashed 1px #000;
}

.close,
.edit {
  display: none;
}
</style>
<style lang="scss">
.lq-active-class {
  .handle.handle-mr {
    border-radius: 50%;
    width: 20px;
    height: 20px;
    margin-top: -10px;
  }

  .handle-mr::before {
    content: " ";
    width: 0;
    height: 0;
    border-top: 7px transparent solid;
    border-bottom: 7px transparent solid;
    border-right: 6px solid rgb(122, 121, 121);
    position: absolute;
    left: 2px;
    top: 2px;
  }

  .handle-mr::after {
    content: " ";
    width: 0;
    height: 0;
    border-top: 7px transparent solid;
    border-bottom: 7px transparent solid;
    border-left: 6px solid rgb(122, 121, 121);
    position: absolute;
    right: 2px;
    top: 2px;
  }

  .close {
    display: flex !important;
    position: absolute;
    right: -10px;
    top: -15px;
    color: red;
    z-index: 9999;
  }

  .edit {
    display: flex !important;
    position: absolute;
    right: 5px;
    top: -15px;
    color: red;
    z-index: 9999;
  }
}

.color-picker {
  .el-color-picker__icon {
    display: none !important;
  }

  .el-icon-arrow-down {
    display: none !important;
  }

  .el-color-picker__color-inner {
    position: relative !important;
    height: 20% !important;
    bottom: 0;
    display: block !important;
    border-top: 1px solid;
    border-bottom: 1px solid;
  }
}

.font-color-picker {
  .el-color-picker__color::before {
    content: "A";
    color: #181515;
    font-size: 17px;
  }
}

.back-color-picker {
  .el-color-picker__color::before {
    display: block;
    height: 80%;
    content: "B";
    background-size: 100% 100%;
  }
}

.cube {
  display: inline-block;
  border-radius: 5px;
  width: 50px;
  height: 40px;
  margin: 4px;
  background: #eee;
  --color: #4af;
}

h1,
.description {
  text-align: center;
}

.button {
  border: 1px solid #333;
  color: #333;
  background: transparent;
  appearance: none;
  -webkit-appearance: none;
  box-sizing: border-box;
  cursor: pointer;
  width: 120px;
  height: 42px;
  font-size: 14px;
  letter-spacing: 1px;
  transition: all ease 0.2s;
  margin: 0px 5px;
}

.button:hover {
  background: #333;
  color: white;
}

.elements {
  //margin-top: 40px;
  border: 2px solid #eee;
}

.selecto-area {
  padding: 20px;
}

#selecto1 .cube {
  transition: all ease 0.2s;
}

.moveable #selecto1 .cube {
  transition: none;
}

.selecto-area .selected {
  color: #fff;
  background: var(--color);
}

.scroll {
  overflow: auto;
  padding-top: 10px;
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
}

.infinite-viewer,
.scroll {
  width: 100%;
  height: 100%;
  box-sizing: border-box;
}

.infinite-viewer .viewport {
  padding-top: 10px;
}

.empty.elements {
  border: none;
}
</style>
