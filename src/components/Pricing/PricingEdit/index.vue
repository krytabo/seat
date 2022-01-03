<template>
  <div class="home">
    <div class="fixed top-0 w-full z-50">
      <Header></Header>
      <div class="bg-white hidden md:flex flex-col md:flex-row md:ml-auto mt-3 md:mt-0 shadow-md h-12 items-center px-5" id="navbar-collapse">
        <p class="text-xl font-bold">定價表編輯</p>
        <div class="float-right ml-auto">
          <router-link to="/test" class="mr-3">
            <el-button size="small" type="primary">測試</el-button>
          </router-link>
          <el-button size="small" @click="Cancel_Form">取消</el-button>

          <el-button size="small" type="primary" @click="Save_Alert">存檔</el-button>
        </div>
      </div>
    </div>
    <div class="p-12 pt-40 px-40">
      <el-card class="box-card card mb-5">
        <div class="flex card-header bar-primary">
          <span class="card-title">定價表</span>
        </div>

        <section v-if="errored">
          <p>We're sorry, we're not able to retrieve this information at the moment, please try back later</p>
        </section>

        <section v-else>
          <div v-if="loading">Loading...</div>

          <div v-else :key="i" v-for="(currency, i) in info" class="currency">
            {{ currency.name }}:
            <!--            <span class="lighten">
              <span v-html="currency.name"></span>
              {{ currency.rate_float | currencydecimal }}
            </span>-->
          </div>
        </section>

        <div class="card-body h-full">
          <el-form ref="form" :model="form" label-width="90px">
            <el-form-item label="定價表名稱">
              <el-input v-model="form.name"></el-input>
            </el-form-item>
            <el-form-item label="配置圖">
              <el-select v-model="form.region" placeholder="請選擇配置圖" class="w-full">
                <el-option label="Ａ廳" value="A"></el-option>
                <el-option label="Ｂ廳" value="B"></el-option>
              </el-select>
            </el-form-item>
          </el-form>
        </div>
      </el-card>

      <el-card class="box-card card">
        <div class="flex card-header bar-primary">
          <span class="card-title">區域列表</span>
          <div class="ml-auto float-right">
            <el-button size="small" type="primary" @click="AddNewData">新增</el-button>
            <el-button class="mr-5" type="danger" size="small" @click="Delete_Alert">刪除</el-button>
          </div>
        </div>

        <div class="card-body h-full">
          <el-table :data="pricing" ref="multipleTable">
            <el-table-column type="selection" width="55"></el-table-column>
            <!--            <el-table-column label="ID" prop="id"></el-table-column>-->
            <el-table-column label="座位區域" prop="name"></el-table-column>
            <el-table-column label="席數" prop="seat"></el-table-column>
            <el-table-column label="票價" prop="money"></el-table-column>
            <el-table-column label="保留" prop="keep">
              <template slot-scope="scope">
                <el-switch v-model="scope.row.keep" :active-value="true" :inactive-value="false" inactive-color="#B9B9B9" />
              </template>
            </el-table-column>
            <el-table-column label="操作" width="100">
              <template slot-scope="scope">
                <el-button type="text" size="small" @click="delete_pricingList">
                  <p class="text-red-500">刪除</p>
                </el-button>
                <el-button type="text" size="small" @click="getData(scope.$index, scope.row)">編輯</el-button>
              </template>
            </el-table-column>
          </el-table>
          <!--          <p>{{ selected }}</p>

          <div id="droptarget" @dragover.prevent="allowDrop" @drop.prevent="drop">
            {{ selectEndList }}
          </div>
          <p id="text"></p>

          <vue-selecto
            dragContainer=".elements"
            v-bind:selectableTargets="['.selecto-area .cube']"
            v-bind:hitRate="100"
            v-bind:selectByClick="true"
            v-bind:selectFromInside="true"
            v-bind:continueSelect="true"
            v-bind:ratio="0"
            @select="onSelect"
            @selectEnd="selectEnd"
          ></vue-selecto>

          <div class="elements selecto-area" id="selecto1">
            <div class="cube" v-for="(cube, index) in cubes" :key="index" :id="index + 1">{{ index + 1 }}</div>
          </div>-->
        </div>
      </el-card>

      <!-- ==================== -->
      <!--        新增區域       -->
      <!-- ==================== -->
      <el-dialog :title="titleMap[dialogStatus]" :visible.sync="Add_Area_Dialog" width="60%" :before-close="handleClose" :close-on-click-modal="close_modal">
        <span>
          <el-form ref="form" :model="Add_Area" label-width="90px" class="text-left">
            <el-form-item label="區域名稱">
              <el-input v-model="Add_Area.area_name" placeholder="請輸入區域名稱"></el-input>
            </el-form-item>
            <el-form-item label="區域定價">
              <el-input-number v-model="Add_Area.area_money" :min="1" label="請輸入價格"></el-input-number>
            </el-form-item>
            <el-form-item label="區域保留">
              <el-switch v-model="Add_Area.area_keep"></el-switch>
            </el-form-item>
          </el-form>
          <p>共有{{ this.cubes.length }}個位置,{{ none === 0 ? "已全選" : "已選擇" + (this.cubes.length - none) + "個,未選擇" + none + "個" }}</p>
          <vue-selecto
            dragContainer=".elements"
            v-bind:selectableTargets="['.selecto-area .cube']"
            v-bind:hitRate="100"
            v-bind:selectByClick="true"
            v-bind:selectFromInside="true"
            v-bind:continueSelect="true"
            v-bind:ratio="0"
            @select="onSelect"
          ></vue-selecto>
          <div class="elements selecto-area relative w-full h-screen overflow-scroll text-left cursor-crosshair" id="selecto1">
            <div
              class="cube text-black rounded-lg text-center cursor-pointer absolute border-white border-2"
              v-for="(item, index) in cubes"
              v-bind:key="index"
              :ref="cubes"
              v-bind:style="{
                color: cubes[index].textcolor,
                background: cubes[index].backgroundColor,
                left: cubes[index].x + 'px',
                top: cubes[index].y + 'px',
                height: cubes[index].height + 'px',
                width: cubes[index].width + 'px',
              }"
              v-bind:class="[cubes[index].status ? 'selected' : ' ']"
              @click="chaneStatus"
              :id="'seat.' + item.customId"
            >
              <!--              v-bind:class="[cubes[index].status ? 'selected' : ' ']"-->
              {{ item.status }}
            </div>
            <!--            <div
              v-for="(item, index) in cubes"
              :key="index"
              :ref="cubes"
              v-bind:style="{
                color: cubes[index].textcolor,
                background: cubes[index].backgroundColor,
                left: cubes[index].x + 'px',
                top: cubes[index].y + 'px',
                height: cubes[index].height + 'px',
                width: cubes[index].width + 'px',
              }"
              class="cube text-black rounded-lg text-center cursor-pointer absolute border-white border-2"
              :class="{ selected: isTransform }"
              type="checkbox"
              @click="changeValue(item)"
              :id="'seat.' + item.customId"
            >
              <p class="grid items-center h-full">{{ item.name }}</p>
            </div>-->
          </div>
        </span>
        <!--        <div v-for="index in selectedLabel">
          <el-input size="mini" v-model="index.customId"></el-input>
        </div>-->
        <span slot="footer" class="dialog-footer">
          <el-button @click="Cancel_Alert">取 消</el-button>
          <el-button type="primary" @click="Add_pricingList">存 檔</el-button>
        </span>
      </el-dialog>
    </div>
  </div>
</template>

<script>
import Header from "@/components/Header";
import { VueSelecto } from "vue-selecto";
import _ from "lodash";
import axios from "axios";

export default {
  name: "PricingEdit",
  components: {
    Header,
    VueSelecto,
  },
  data() {
    return {
      close_modal: false, //禁止點dialog框外關閉視窗

      color: "#333",
      colors: {
        color: "red",
        bg: "#333333",
      },

      number: 1, //金額起始值
      Add_Area_Dialog: false, //編輯區域視窗

      //定價表格
      form: {
        name: "",
        region: "",
      },

      //新增區域表格
      Add_Area: {
        area_name: "",
        area_seat: "",
        area_money: "",
        area_keep: false,
      },

      // 座位表
      cubes: [
        {
          customId: "0",
          name: "控制室",
          x: 280,
          y: 0,
          textcolor: "#333333",
          backgroundColor: "#FFCBCB",
          height: 40,
          width: 255,
          status: false,
        },
        {
          customId: "241",
          name: "舞台",
          x: 120,
          y: 680,
          textcolor: "#333333",
          backgroundColor: "#FFCBCB",
          height: 80,
          width: 570,
          status: false,
        },
        {
          customId: "1",
          name: "1",
          x: 20,
          y: 80,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "2",
          name: "2",
          x: 60,
          y: 80,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "3",
          name: "3",
          x: 100,
          y: 80,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "4",
          name: "4",
          x: 140,
          y: 80,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "5",
          name: "5",
          x: 20,
          y: 120,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "6",
          name: "6",
          x: 60,
          y: 120,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "7",
          name: "7",
          x: 100,
          y: 120,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "8",
          name: "8",
          x: 140,
          y: 120,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "9",
          name: "9",
          x: 20,
          y: 160,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "10",
          name: "10",
          x: 60,
          y: 160,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "11",
          name: "11",
          x: 100,
          y: 160,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "12",
          name: "12",
          x: 140,
          y: 160,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "13",
          name: "13",
          x: 20,
          y: 200,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "14",
          name: "14",
          x: 60,
          y: 200,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "15",
          name: "15",
          x: 100,
          y: 200,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "16",
          name: "16",
          x: 140,
          y: 200,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "17",
          name: "17",
          x: 20,
          y: 240,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "18",
          name: "18",
          x: 60,
          y: 240,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "19",
          name: "19",
          x: 100,
          y: 240,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "20",
          name: "20",
          x: 140,
          y: 240,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "21",
          name: "21",
          x: 20,
          y: 300,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "22",
          name: "22",
          x: 60,
          y: 300,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "23",
          name: "23",
          x: 100,
          y: 300,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "24",
          name: "24",
          x: 140,
          y: 300,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "25",
          name: "25",
          x: 20,
          y: 340,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "26",
          name: "26",
          x: 60,
          y: 340,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "27",
          name: "27",
          x: 100,
          y: 340,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "28",
          name: "28",
          x: 140,
          y: 340,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "29",
          name: "29",
          x: 20,
          y: 380,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "30",
          name: "30",
          x: 60,
          y: 380,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "31",
          name: "31",
          x: 100,
          y: 380,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "32",
          name: "32",
          x: 140,
          y: 380,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "33",
          name: "33",
          x: 20,
          y: 420,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "34",
          name: "34",
          x: 60,
          y: 420,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "35",
          name: "35",
          x: 100,
          y: 420,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "36",
          name: "36",
          x: 140,
          y: 420,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "37",
          name: "37",
          x: 20,
          y: 460,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "38",
          name: "38",
          x: 60,
          y: 460,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "39",
          name: "39",
          x: 100,
          y: 460,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "40",
          name: "40",
          x: 140,
          y: 460,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "41",
          name: "41",
          x: 20,
          y: 500,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "42",
          name: "42",
          x: 60,
          y: 500,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "43",
          name: "43",
          x: 100,
          y: 500,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "44",
          name: "44",
          x: 140,
          y: 500,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "45",
          name: "45",
          x: 20,
          y: 540,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "46",
          name: "46",
          x: 60,
          y: 540,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "47",
          name: "47",
          x: 100,
          y: 540,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "48",
          name: "48",
          x: 140,
          y: 540,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "49",
          name: "49",
          x: 40,
          y: 580,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "50",
          name: "50",
          x: 80,
          y: 580,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "51",
          name: "51",
          x: 120,
          y: 580,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "52",
          name: "52",
          x: 40,
          y: 620,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "53",
          name: "53",
          x: 80,
          y: 620,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "54",
          name: "54",
          x: 120,
          y: 620,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "55",
          name: "55",
          x: 240,
          y: 80,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "56",
          name: "56",
          x: 280,
          y: 80,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "57",
          name: "57",
          x: 320,
          y: 80,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "58",
          name: "58",
          x: 360,
          y: 80,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "59",
          name: "59",
          x: 400,
          y: 80,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "60",
          name: "60",
          x: 440,
          y: 80,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "61",
          name: "61",
          x: 480,
          y: 80,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "62",
          name: "62",
          x: 520,
          y: 80,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "63",
          name: "63",
          x: 220,
          y: 120,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "64",
          name: "64",
          x: 260,
          y: 120,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "65",
          name: "65",
          x: 300,
          y: 120,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "66",
          name: "66",
          x: 340,
          y: 120,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "67",
          name: "67",
          x: 380,
          y: 120,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "68",
          name: "68",
          x: 420,
          y: 120,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "69",
          name: "69",
          x: 460,
          y: 120,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "70",
          name: "70",
          x: 500,
          y: 120,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "71",
          name: "71",
          x: 540,
          y: 120,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "72",
          name: "72",
          x: 220,
          y: 160,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "73",
          name: "73",
          x: 260,
          y: 160,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "74",
          name: "74",
          x: 300,
          y: 160,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "75",
          name: "75",
          x: 340,
          y: 160,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "76",
          name: "76",
          x: 380,
          y: 160,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "77",
          name: "77",
          x: 420,
          y: 160,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "78",
          name: "78",
          x: 460,
          y: 160,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "79",
          name: "79",
          x: 500,
          y: 160,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "80",
          name: "80",
          x: 540,
          y: 160,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "81",
          name: "81",
          x: 220,
          y: 200,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "82",
          name: "82",
          x: 260,
          y: 200,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "83",
          name: "83",
          x: 300,
          y: 200,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "84",
          name: "84",
          x: 340,
          y: 200,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "85",
          name: "85",
          x: 380,
          y: 200,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "86",
          name: "86",
          x: 420,
          y: 200,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "87",
          name: "87",
          x: 460,
          y: 200,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "88",
          name: "88",
          x: 500,
          y: 200,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "89",
          name: "89",
          x: 540,
          y: 200,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "90",
          name: "90",
          x: 220,
          y: 240,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "91",
          name: "91",
          x: 260,
          y: 240,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "92",
          name: "92",
          x: 300,
          y: 240,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "93",
          name: "93",
          x: 340,
          y: 240,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "94",
          name: "94",
          x: 380,
          y: 240,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "95",
          name: "95",
          x: 420,
          y: 240,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "96",
          name: "96",
          x: 460,
          y: 240,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "97",
          name: "97",
          x: 500,
          y: 240,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "98",
          name: "98",
          x: 540,
          y: 240,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "99",
          name: "99",
          x: 220,
          y: 300,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "100",
          name: "100",
          x: 260,
          y: 300,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "101",
          name: "101",
          x: 300,
          y: 300,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "102",
          name: "102",
          x: 340,
          y: 300,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "103",
          name: "103",
          x: 380,
          y: 300,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "104",
          name: "104",
          x: 420,
          y: 300,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "105",
          name: "105",
          x: 460,
          y: 300,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "106",
          name: "106",
          x: 500,
          y: 300,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "107",
          name: "107",
          x: 540,
          y: 300,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "108",
          name: "108",
          x: 220,
          y: 340,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "109",
          name: "109",
          x: 260,
          y: 340,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "110",
          name: "110",
          x: 300,
          y: 340,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "111",
          name: "111",
          x: 340,
          y: 340,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "112",
          name: "112",
          x: 380,
          y: 340,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "113",
          name: "113",
          x: 420,
          y: 340,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "114",
          name: "114",
          x: 460,
          y: 340,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "115",
          name: "115",
          x: 500,
          y: 340,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "116",
          name: "116",
          x: 540,
          y: 340,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "117",
          name: "117",
          x: 220,
          y: 380,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "118",
          name: "118",
          x: 260,
          y: 380,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "119",
          name: "119",
          x: 300,
          y: 380,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "120",
          name: "120",
          x: 340,
          y: 380,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "121",
          name: "121",
          x: 380,
          y: 380,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "122",
          name: "122",
          x: 420,
          y: 380,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "123",
          name: "123",
          x: 460,
          y: 380,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "124",
          name: "124",
          x: 500,
          y: 380,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "125",
          name: "125",
          x: 540,
          y: 380,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "126",
          name: "126",
          x: 220,
          y: 420,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "127",
          name: "127",
          x: 260,
          y: 420,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "128",
          name: "128",
          x: 300,
          y: 420,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "129",
          name: "129",
          x: 340,
          y: 420,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "130",
          name: "130",
          x: 380,
          y: 420,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "131",
          name: "131",
          x: 420,
          y: 420,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "132",
          name: "132",
          x: 460,
          y: 420,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "133",
          name: "133",
          x: 500,
          y: 420,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "134",
          name: "134",
          x: 540,
          y: 420,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "135",
          name: "135",
          x: 220,
          y: 460,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "136",
          name: "136",
          x: 260,
          y: 460,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "137",
          name: "137",
          x: 300,
          y: 460,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "138",
          name: "138",
          x: 340,
          y: 460,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "139",
          name: "139",
          x: 380,
          y: 460,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "140",
          name: "140",
          x: 420,
          y: 460,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "141",
          name: "141",
          x: 460,
          y: 460,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "142",
          name: "142",
          x: 500,
          y: 460,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "143",
          name: "143",
          x: 540,
          y: 460,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "144",
          name: "144",
          x: 220,
          y: 500,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "145",
          name: "145",
          x: 260,
          y: 500,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "146",
          name: "146",
          x: 300,
          y: 500,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "147",
          name: "147",
          x: 340,
          y: 500,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "148",
          name: "148",
          x: 380,
          y: 500,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "149",
          name: "149",
          x: 420,
          y: 500,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "150",
          name: "150",
          x: 460,
          y: 500,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "151",
          name: "151",
          x: 500,
          y: 500,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "152",
          name: "152",
          x: 540,
          y: 500,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "153",
          name: "153",
          x: 220,
          y: 540,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "154",
          name: "154",
          x: 260,
          y: 540,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "155",
          name: "155",
          x: 300,
          y: 540,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "156",
          name: "156",
          x: 340,
          y: 540,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "157",
          name: "157",
          x: 380,
          y: 540,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "158",
          name: "158",
          x: 420,
          y: 540,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "159",
          name: "159",
          x: 460,
          y: 540,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "160",
          name: "160",
          x: 500,
          y: 540,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "161",
          name: "161",
          x: 540,
          y: 540,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "162",
          name: "162",
          x: 240,
          y: 580,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "163",
          name: "163",
          x: 280,
          y: 580,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "164",
          name: "164",
          x: 320,
          y: 580,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "165",
          name: "165",
          x: 360,
          y: 580,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "166",
          name: "166",
          x: 400,
          y: 580,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "167",
          name: "167",
          x: 440,
          y: 580,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "168",
          name: "168",
          x: 480,
          y: 580,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "169",
          name: "169",
          x: 520,
          y: 580,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "170",
          name: "170",
          x: 240,
          y: 620,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "171",
          name: "171",
          x: 280,
          y: 620,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "172",
          name: "172",
          x: 320,
          y: 620,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "173",
          name: "173",
          x: 360,
          y: 620,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "174",
          name: "174",
          x: 400,
          y: 620,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "175",
          name: "175",
          x: 440,
          y: 620,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "176",
          name: "176",
          x: 480,
          y: 620,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "177",
          name: "177",
          x: 520,
          y: 620,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "178",
          name: "178",
          x: 640,
          y: 80,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "179",
          name: "179",
          x: 680,
          y: 80,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "180",
          name: "180",
          x: 720,
          y: 80,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "181",
          name: "181",
          x: 760,
          y: 80,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "182",
          name: "182",
          x: 640,
          y: 120,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "183",
          name: "183",
          x: 680,
          y: 120,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "184",
          name: "184",
          x: 720,
          y: 120,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "185",
          name: "185",
          x: 760,
          y: 120,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "186",
          name: "186",
          x: 640,
          y: 160,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "187",
          name: "187",
          x: 680,
          y: 160,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "188",
          name: "188",
          x: 720,
          y: 160,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "189",
          name: "189",
          x: 760,
          y: 160,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "190",
          name: "190",
          x: 640,
          y: 200,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "191",
          name: "191",
          x: 680,
          y: 200,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "192",
          name: "192",
          x: 720,
          y: 200,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "193",
          name: "193",
          x: 760,
          y: 200,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "194",
          name: "194",
          x: 640,
          y: 240,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "195",
          name: "195",
          x: 680,
          y: 240,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "196",
          name: "196",
          x: 720,
          y: 240,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "197",
          name: "197",
          x: 760,
          y: 240,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "198",
          name: "198",
          x: 640,
          y: 300,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "199",
          name: "199",
          x: 680,
          y: 300,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "200",
          name: "200",
          x: 720,
          y: 300,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "201",
          name: "201",
          x: 760,
          y: 300,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "202",
          name: "202",
          x: 640,
          y: 340,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "203",
          name: "203",
          x: 680,
          y: 340,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "204",
          name: "204",
          x: 720,
          y: 340,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "205",
          name: "205",
          x: 760,
          y: 340,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "206",
          name: "206",
          x: 640,
          y: 380,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "207",
          name: "207",
          x: 680,
          y: 380,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "208",
          name: "208",
          x: 720,
          y: 380,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "209",
          name: "209",
          x: 760,
          y: 380,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "210",
          name: "210",
          x: 640,
          y: 420,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "211",
          name: "211",
          x: 680,
          y: 420,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "212",
          name: "212",
          x: 720,
          y: 420,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "213",
          name: "213",
          x: 760,
          y: 420,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "214",
          name: "214",
          x: 640,
          y: 460,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "215",
          name: "215",
          x: 680,
          y: 460,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "216",
          name: "216",
          x: 720,
          y: 460,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "217",
          name: "217",
          x: 760,
          y: 460,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "218",
          name: "218",
          x: 640,
          y: 500,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "219",
          name: "219",
          x: 680,
          y: 500,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "220",
          name: "220",
          x: 720,
          y: 500,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "221",
          name: "221",
          x: 760,
          y: 500,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "222",
          name: "222",
          x: 640,
          y: 540,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "223",
          name: "223",
          x: 680,
          y: 540,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "224",
          name: "224",
          x: 720,
          y: 540,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "225",
          name: "225",
          x: 760,
          y: 540,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "226",
          name: "226",
          x: 660,
          y: 580,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "227",
          name: "227",
          x: 700,
          y: 580,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "228",
          name: "227",
          x: 740,
          y: 580,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "229",
          name: "229",
          x: 660,
          y: 620,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "230",
          name: "230",
          x: 700,
          y: 620,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "231",
          name: "231",
          x: 740,
          y: 620,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "232",
          name: "232",
          x: 860,
          y: 460,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "233",
          name: "233",
          x: 900,
          y: 460,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "234",
          name: "234",
          x: 860,
          y: 500,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "235",
          name: "235",
          x: 900,
          y: 500,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "236",
          name: "236",
          x: 860,
          y: 540,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "237",
          name: "237",
          x: 900,
          y: 540,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "238",
          name: "238",
          x: 860,
          y: 580,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "239",
          name: "239",
          x: 900,
          y: 580,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
        {
          customId: "240",
          name: "240",
          x: 60,
          y: 40,
          textcolor: "#333333",
          backgroundColor: "#eeeeee",
          height: 40,
          width: 40,
          status: false,
        },
      ],

      // 區域列表
      pricing: [
        {
          id: "12345",
          name: "A區",
          seat: "20",
          money: "1200",
          keep: true,
        },
      ],
      selectedLabel: [],

      titleMap: {
        addEquipment: "新增區域",
        editEquipment: "編輯區域",
      },
      //新增和編輯的標題
      dialogStatus: "",

      info: null,
      loading: true,
      errored: false,
    };
  },
  filters: {
    currencydecimal(value) {
      return value.toFixed(2);
    },
  },
  mounted() {
    /*    for (let i = 0; i < 60; ++i) {
      this.cubes.push(i);
    }*/
    axios
      // .get("https://api.coindesk.com/v1/bpi/currentprice.json")
      .get("http://localhost:8080/#/api/seaData.js")
      .then((response) => {
        this.info = response.data.customId;
      })
      .catch((error) => {
        console.log(error);
        this.errored = true;
      })
      .finally(() => (this.loading = false));
  } /*自動載入函式*/,
  created() {},
  computed: {
    none() {
      return this.cubes.filter(function (val) {
        return val.status === false;
      }).length;
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
            this.Add_Area_Dialog = false;
          }
        });
    },

    // form取消
    Cancel_Form() {
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
            window.location.href = "#/";
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
            window.location.href = "#/";
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

    // dialog存擋
    Add_pricingList() {
      this.$swal
        .fire({
          title: "存檔完成！",
          icon: "success",
          confirmButtonColor: "#0084ff",
          showConfirmButton: false,
          timer: 900,
        })
        .then(() => {
          this.Add_Area_Dialog = false;
          this.pricing.push({
            name: this.Add_Area.area_name,
            seat: this.Add_Area.area_seat,
            money: this.Add_Area.area_money,
            keep: this.Add_Area.area_keep,
          });
          this.fileList.push([]);
        });
    },

    // =======其他=======

    // 新增檔案
    AddNewData() {
      this.Add_Area_Dialog = true;
      this.dialogStatus = "addEquipment";
    },
    //讀取檔案
    getData(index, row) {
      /*this.$axios
        .get("@/static/data.json")
        .then((res) => {
          console.log(res);
        })
        .catch((error) => {
          console.log(error);
        });*/
      this.Add_Area_Dialog = true;
      this.Add_Area = row;
      console.log(index, row);
      this.dialogStatus = "editEquipment";
    },

    // 選擇區域
    onSelect(e) {
      let self = this;
      e.added.forEach((el) => {
        el.classList.add("selected");
        let customId = el.id.split("seat.")[1];
        let data = _.find(self.cubes, ["customId", customId]);
        this.chaneStatus(data);
        console.log(data);
      });

      e.removed.forEach((el) => {
        el.classList.remove("selected");
        let customId = el.id.split("seat.")[1];
        let data = _.find(self.cubes, ["customId", customId]);
        this.chaneStatus(data);
      });
    },

    chaneStatus(item) {
      if (item.status == false) {
        item.status = true;
        this.selectedLabel.push(item.val);
      } else {
        item.status = false;
        this.selectedLabel.splice(item, 1);
      }
    },

    // 刪除定價列表
    delete_pricingList(i) {
      this.pricing.splice(i, 1);
      // 删除的时候要把fileList清除,否则页面已上传的文件不会被清空
      this.fileList.splice(i, 1);
      console.log(this.fileList);
    },
  },
};
</script>

<style scoped lang="scss">
.firsty {
  width: 180px;
  height: 60px;
  text-align: center;
  line-height: 60px;
  color: #333;
  font-size: 28px;
  border-radius: 8px;
  border: 1px solid #eeeeee;
  margin-bottom: 20px;
}

.active {
  background-color: #61b1f3;
  color: #fff;
}

#selecto1 .cube {
  transition: all ease 0.2s;
}

.moveable #selecto1 .cube {
  transition: none;
}

/*.selecto-area .selected {
  color: #fff !important;
  background: var(--color) !important;
}*/

.selected {
  color: #fff !important;
  background: var(--color) !important;
}
</style>
