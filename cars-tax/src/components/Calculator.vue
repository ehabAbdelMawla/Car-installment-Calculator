<template>
  <Header />

  <div class="inputs-container wow fadeIn" id="capture">
    <article :class="{ result: true, wow: true, fadeInDown: true }">
      <div class="form-group">
        <label for="car-price">سعر السيارة</label>
        <input id="car-price" type="number" placeholder="0" v-model="price" />
      </div>
      <div class="form-group">
        <label for="car-down-payment-value">قيمة المقدم</label>
        <input
          id="car-down-payment-value"
          :type="isAndroid ? 'tel' : 'number'"
          placeholder="0"
          v-model="downpaymentValue"
          :readonly="price == 0"
        />
      </div>
      <div class="form-group">
        <label for="car-down-payment-percent">نسبة المقدم</label>
        <input
          id="car-down-payment-percent"
          :type="isAndroid ? 'tel' : 'number'"
          placeholder="0"
          v-model="downpaymentPercent"
          :readonly="price == 0"
        />
      </div>
      <div class="form-group" data-html2canvas-ignore="true">
        <label for="car-down-payment-percent">نسبة الفائدة</label>
        <input
          id="car-down-payment-percent"
          type="number"
          placeholder="0"
          v-model="additions"
          :readonly="price == 0"
        />
      </div>
    </article>
    <div :class="{ result: true, wow: true, fadeInDown: true }">
      <div>
        <p>قيمة التمويل</p>
        <p>{{ remain }}</p>
      </div>
      <div>
        <p>مصاريف ادارية</p>
        <p>{{ masreefEdaria }}</p>
      </div>
      <div>
        <p>تأمين</p>
        <p>{{ taminat }}</p>
      </div>
    </div>
    <div :class="{ result: true, wow: true, fadeInDown: true }">
      <div v-for="num in 7" :key="num">
        <p>{{ getYearsSuffix(num) }}</p>
        <p>{{ taxPerYear(num) }}</p>
      </div>
    </div>
  </div>
  <button
    class="sreenBtn"
    @click="takeScreenShot"
    data-html2canvas-ignore="true"
  >
    <i class="fa fa-camera-retro" />
  </button>
</template>

<script>
import "../styles/Calculator.css";
import html2canvas from "html2canvas";
import swal from "sweetalert";
import { Filesystem, Directory } from "@capacitor/filesystem";
import Header from "./Header.vue";
import { App } from "@capacitor/app";
export default {
  name: "Calculator",
  components: {
    Header,
  },
  data() {
    return {
      price: "",
      downpaymentValue: "",
      downpaymentPercent: "",
      additions: "",
      percentWithoutFormat: "",
      isAndroid: /Android/i.test(navigator.userAgent),
    };
  },
  created() {
    document.addEventListener("backbutton", async function() {
      App.exitApp();
    });
  },
  methods: {
    shortFormat(number) {
      return Math.ceil(Number(number));
    },
    getYearsSuffix(num) {
      return num == 1 ? "سنة" : num == 2 ? "سنتين" : num + " سنين";
    },
    takeScreenShot() {
      if (this.price > 0) {
        swal("Type Image Name:", {
          content: "input",
        }).then((value) => {
          if (value || value == "") {
            window.scrollTo(0, 0);
            html2canvas(document.getElementById("capture")).then(
              async (canvas) => {
                if (this.isAndroid) {
                  // .... Android ....

                  await Filesystem.writeFile({
                    path: value == "" ? `${Date.now()}.png` : `${value}.png`,
                    data: canvas.toDataURL(),
                    directory: Directory.Documents,
                  });
                } else {
                  //    .... Web ....
                  const aTag = document.createElement("a");
                  aTag.setAttribute("href", canvas.toDataURL());
                  aTag.setAttribute(
                    "download",
                    value == "" ? "image.png" : value
                  );
                  aTag.click();
                }
                swal("Image Saved.", {
                  icon: "success",
                });
              }
            );
          }
        });
      }
    },
  },
  watch: {
    price(newValue) {
      if (newValue > 0) {
        if (this.downpaymentValue > 0) {
          this.percentWithoutFormat = (this.downpaymentValue / newValue) * 100;
          this.downpaymentPercent = this.shortFormat(this.percentWithoutFormat);
        } else if (this.downpaymentPercent > 0) {
          this.downpaymentValue = this.shortFormat(
            (this.downpaymentPercent / 100) * this.price
          );
        }
      }
    },
    downpaymentValue(newValue) {
      if (this.price > 0) {
        this.percentWithoutFormat = (newValue / this.price) * 100;
        this.downpaymentPercent = this.shortFormat(this.percentWithoutFormat);
      } else {
        this.downpaymentPercent = 0;
      }
    },
    downpaymentPercent(newValue) {
      if (this.shortFormat(this.percentWithoutFormat) != newValue) {
        this.percentWithoutFormat = newValue;
      }
      if (this.price > 0) {
        this.downpaymentValue = this.shortFormat(
          (this.percentWithoutFormat / 100) * this.price
        );
      } else {
        this.downpaymentValue = 0;
      }
    },
  },
  computed: {
    remain() {
      return this.shortFormat(
        this.shortFormat(this.price - this.downpaymentValue)
      );
    },
    masreefEdaria() {
      return this.shortFormat(this.remain * 0.0249);
    },
    taminat() {
      return this.shortFormat(this.price * 0.025);
    },
    taxPerYear() {
      return (numOfYear) => {
        const additions = Number(this.additions);
        return this.shortFormat(
          this.shortFormat(
            (((numOfYear * additions) / 100) * this.remain + this.remain) /
              (numOfYear * 12)
          )
        );
      };
    },
  },
};
</script>
