<template>
  <div class="hello">
    <el-form>
      <el-form-item>
        加密方式:
        <el-select v-model="value" clearable placeholder="请选择加密方式">
          <el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item class="form">
        明文:
        <el-input placeholder="请输入英文大小写字母~" v-model="input1" type="textarea"></el-input>
      </el-form-item>
      <el-from-item class="input2">
        密钥:
        <el-input size="mini" placeholder="请输入密钥" v-model="input2" clearable>
        </el-input>
      </el-from-item>
      <el-form-item>
        密文:
        <el-input placeholder="显示最终结果~" v-model="input3" type="textarea"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button size="mini" type="primary" @click="encrypt">加密</el-button>
        <el-button size="mini" type="primary" @click="decrypt">解密</el-button>
        <label for="file-upload-input" class="custom-el-button">选择文件</label>
        <input type="file" id="file-upload-input" @change="handleFileUpload" accept=".txt" style="display: none;">
        <el-button size="mini" type="primary" @click="generateChart">生成统计图表</el-button>
        <el-button size="mini" @click="clear">清空</el-button>
      </el-form-item>
    </el-form>
    <div ref="myChart" style="width: 600px;height:400px;"></div>
  </div>
</template>

<script>
import * as echarts from 'echarts';
export default {
  data() {
    return {
      input1: '',
      input2: '',
      input3: '',
      options: [{
        value: '位移加密',
        label: '位移加密'
      }, {
        value: '仿射加密',
        label: '仿射加密'
      }, {
        value: '置换加密',
        label: '置换加密'
      }, {
        value: '流密码',
        label: '流密码'
      }],
      value: '',
    };
  },
  methods: {
    encrypt() {
      if (this.value == "位移加密") {
        this.jiami();
      }
      if (this.value == "仿射加密") {
        this.affineEncrypt();
      }
      if (this.value == "置换加密") {
        this.encryptMessage();
      }
      if (this.value == "流密码") {
        this.streamCipher(this.input1, true);
      }
    },
    decrypt() {
      if (this.value == "位移加密") {
        this.jiemi();
      }
      if (this.value == "仿射加密") {
        this.affineDecrypt();
      }
      if (this.value == "置换加密") {
        this.decryptMessage();
      }
      if (this.value == "流密码") {
        this.streamCipher(this.input1, true);
      }
    },
    clear() {
      this.input1 = '';
      this.input2 = '';
      this.input3 = '';
    },
    jiemi(content) {
      // 假设有一些方式来获取用户输入的 num 和 targetText
      let num = this.input2;
      content = this.input1; // 替换为目标文本
      // 如果 num 是空字符串，则设置为 0，否则将其转换为整数
      num = num === '' ? 0 : parseInt(num);

      // 校验 num 是否在 1 到 25 之间
      if (num < 1 || num > 25) {
        alert("错误: 请输入正确的数据！");
        return;
      }
      let ret = ""; // 解密后的结果
      for (let i = 0; i < content.length; i++) {
        let ch = content[i];
        if (ch >= 'a' && ch <= 'z') {
          // 如果是小写字母，进行偏移
          let offset = ((ch.charCodeAt(0) - 'a'.charCodeAt(0) - num) + 26) % 26;
          let en_char = String.fromCharCode(offset + 'a'.charCodeAt(0));
          ret += en_char;
        } else if (ch >= 'A' && ch <= 'Z') {
          // 如果是大写字母，进行偏移
          let offset = ((ch.charCodeAt(0) - 'A'.charCodeAt(0) - num) + 26) % 26;
          let en_char = String.fromCharCode(offset + 'A'.charCodeAt(0));
          ret += en_char;
        } else {
          // 其他字符不变
          ret += ch;
        }
      }

      this.input3 = ret; // 输出解密后的结果
    },
    jiami(content) {
      // 假设已经有某种方法获取用户输入的 num 和加密文本 targetText
      let num = this.input2;
      content = this.input1; // 替换为加密文本

      // 如果 num 是空字符串，则设置为 0，否则将其转换为整数
      num = num === '' ? 0 : parseInt(num);

      // 校验 num 是否在 1 到 25 之间
      if (num < 1 || num > 25) {
        alert("错误: 请输入正确的数据！");
        return;
      }

      let ret = ""; // 解密后的结果
      for (let i = 0; i < content.length; i++) {
        let ch = content[i];

        if (ch >= 'a' && ch <= 'z') {
          // 如果是小写字母，进行偏移
          let offset = ((ch.charCodeAt(0) - 'a'.charCodeAt(0) + num) % 26);
          let de_char = String.fromCharCode(offset + 'a'.charCodeAt(0));
          ret += de_char;
        } else if (ch >= 'A' && ch <= 'Z') {
          // 如果是大写字母，进行偏移
          let offset = ((ch.charCodeAt(0) - 'A'.charCodeAt(0) + num) % 26);
          let de_char = String.fromCharCode(offset + 'A'.charCodeAt(0));
          ret += de_char;
        } else {
          // 其他字符不变
          ret += ch;
        }
      }

      this.input3 = ret; // 输出解密后的结果
    },
    //仿射加密算法
    affineEncrypt(content) {
      let str = this.input2; // 这是你的输入字符串
      let parts = str.split(','); // 分割字符串

      // 确保数组有两部分，且每部分都是有效的数字
      if (parts.length === 2 && !isNaN(parts[0]) && !isNaN(parts[1])) {
        let a = parseInt(parts[0], 10); // 逗号前的数转化为数字
        let b = parseInt(parts[1], 10); // 逗号后的数转化为数字
        const m = 26; // 英文字母的数量
        content = '';
        for (let char of this.input1) {
          if (char >= 'a' && char <= 'z') {
            let x = char.charCodeAt(0) - 'a'.charCodeAt(0);
            let y = (a * x + b) % m;
            content += String.fromCharCode(y + 'a'.charCodeAt(0));
          } else if (char >= 'A' && char <= 'Z') {
            let x = char.charCodeAt(0) - 'A'.charCodeAt(0);
            let y = (a * x + b) % m;
            content += String.fromCharCode(y + 'A'.charCodeAt(0));
          } else {
            content += char;
          }
        }
        this.input3 = content;
      } else {
        this.message.error("请输入正确格式的数据！");
      }
    },
    modInverse(a, m) {
      // 计算a模m的逆
      for (let x = 1; x < m; x++) {
        if (((a % m) * (x % m)) % m == 1) {
          return x;
        }
      }
      return 1; // 如果不存在，则返回1（理论上不会发生这种情况）
    },
    //仿射解密算法
    affineDecrypt(content) {
      let str = this.input2; // 这是你的输入字符串
      let parts = str.split(','); // 分割字符串

      // 确保数组有两部分，且每部分都是有效的数字
      if (parts.length === 2 && !isNaN(parts[0]) && !isNaN(parts[1])) {
        let a = parseInt(parts[0], 10); // 逗号前的数转化为数字
        let b = parseInt(parts[1], 10); // 逗号后的数转化为数字
        const m = 26;
        content = '';
        let a_inv = this.modInverse(a, m);
        for (let char of this.input1) {
          if (char >= 'a' && char <= 'z') {
            let y = char.charCodeAt(0) - 'a'.charCodeAt(0);
            let x = (a_inv * (y - b + m)) % m;
            content += String.fromCharCode(x + 'a'.charCodeAt(0));
          } else if (char >= 'A' && char <= 'Z') {
            let y = char.charCodeAt(0) - 'A'.charCodeAt(0);
            let x = (a_inv * (y - b + m)) % m;
            content += String.fromCharCode(x + 'A'.charCodeAt(0));
          } else {
            content += char;
          }
        }
        this.input3 = content;
      } else {
        this.message.error("请输入正确格式的数据！");
      }
    },
    //置换加密
    encryptMessage(content) {
      const key = parseInt(this.input2); // 确保key是一个整数
      content = this.input1;
      const ciphertext = new Array(key).fill('');
      const nRow = Math.ceil(content.length / key);
      const messageFilled = content.padEnd(nRow * key, ' ');

      for (let col = 0; col < key; col++) {
        let pointer = col;
        while (pointer < nRow * key) {
          ciphertext[col] += messageFilled[pointer];
          pointer += key;
        }
      }
      this.input3 = ciphertext.join('');
    },
    //置换解密
    decryptMessage(content) {
      const key = parseInt(this.input2); // 确保key是一个整数
      content = this.input1;
      const nRow = Math.ceil(content.length / key);
      const plaintext = new Array(nRow).fill('');
      const ciphertextFilled = content.padEnd(nRow * key, ' ');

      for (let col = 0; col < key; col++) {
        let pointer = col * nRow;
        for (let row = 0; row < nRow; row++) {
          plaintext[row] += ciphertextFilled[pointer + row];
        }
      }

      this.input3 = plaintext.join('').trim();
    },
    triggerFileInput() {
      this.$refs.fileInput.click(); // 触发原生文件输入点击事件
    },
    // 处理文件上传并根据选择的加密方式加密内容
    handleFileUpload(event) {
      const file = event.target.files[0];
      const reader = new FileReader();
      reader.onload = (e) => {
        const content = e.target.result.toLowerCase(); // 转换为小写
        this.input1 = content;
        this.encryptContent(content); // 根据选择的加密方式加密内容
      };
      reader.readAsText(file);
    },

    // 根据选择的加密方式加密内容
    encryptContent(content) {
      switch (this.value) {
        case '位移加密':
          this.jiami(content);
          break;
        case '仿射加密':
          this.affineEncrypt(content);
          break;
        case '置换加密':
          this.encryptMessage(content);
          break;
        case '流密码':
          this.streamCipher(this.input1, true);
          break;
        default:
          alert('请选择一个加密方式');
          return;
      }
      // 在加密函数中设置 this.input3 后调用此函数进行字符统计
      this.countChars(this.input1, this.input3);
    },
    countChars(plainText, encryptedText) {
      const charMap = {};
      'abcdefghijklmnopqrstuvwxyz'.split('').forEach(char => {
        charMap[char] = { plain: 0, encrypted: 0 };
      });

      plainText.replace(/[a-z]/g, (char) => (charMap[char].plain = (charMap[char].plain || 0) + 1));
      encryptedText.replace(/[a-z]/g, (char) => (charMap[char].encrypted = (charMap[char].encrypted || 0) + 1));

      this.drawChart(charMap); // 绘制图表
    },
    generateChart() {
      if (this.input1 && this.input3) {
        this.countChars(this.input1, this.input3);
      } else {
        alert('请先上传文件和进行加密');
      }
    },
    drawChart(charMap) {
      const chartDom = this.$refs.myChart;
      const myChart = echarts.init(chartDom);
      const option = {
        tooltip: {},
        legend: {
          data: ['明文柱状', '密文柱状', '明文折线', '密文折线']
        },
        xAxis: {
          type: 'category',
          data: Object.keys(charMap)
        },
        yAxis: {
          type: 'value'
        },
        series: [
          {
            name: '明文柱状',
            type: 'bar',
            data: Object.values(charMap).map(item => item.plain),
            color: 'blue'
          },
          {
            name: '密文柱状',
            type: 'bar',
            data: Object.values(charMap).map(item => item.encrypted),
            color: 'green'
          },
          {
            name: '明文折线',
            type: 'line',
            data: Object.values(charMap).map(item => item.plain),
            color: 'blue',
            // smooth: true, // 可选，使线条平滑
          },
          {
            name: '密文折线',
            type: 'line',
            data: Object.values(charMap).map(item => item.encrypted),
            color: 'green',
            // smooth: true, // 可选，使线条平滑
          }
        ]
      };
      myChart.setOption(option);
    },
    streamCipher(input, isEncrypt) {
      const seed = parseInt(this.input2); // 使用密钥作为种子
      const length = input.length * 8; // 计算需要的随机位数
      const keyStream = this.lfsr(seed, length); // 生成密钥流
      let output = '';

      for (let i = 0; i < input.length; i++) {
        const inputChar = input.charCodeAt(i);
        const keyChar = parseInt(keyStream.substr(i * 8, 8), 2);
        const encryptedChar = inputChar ^ keyChar; // 明文和密钥流的异或
        output += String.fromCharCode(encryptedChar);
      }

      if (isEncrypt) {
        this.input3 = output; // 设置加密结果
      } else {
        this.input1 = output; // 设置解密结果
      }
    },
    lfsr(seed, length) {
      let state = seed; // 初始种子
      let lfsrOutput = "";
      let bit; // 将生成的新位
      for (let i = 0; i < length; i++) {
        // 假设使用最简单的反馈策略：第一个和最后一个位的异或
        bit = ((state >> 0) ^ (state >> 1)) & 1;
        lfsrOutput += bit; // 将新位添加到输出
        state = (state >> 1) | (bit << (seed.toString(2).length - 1)); // 更新状态
      }

      return lfsrOutput;
    }
  }
}
</script>
<style scoped>
.hello {
  width: 100%;
  height: 120vh;
  display: flex;
  /* 启用 flex 布局 */
  justify-content: center;
  /* 水平居中 */

}

.custom-el-button {
  display: inline-block;
  line-height: 1.1;
  white-space: nowrap;
  cursor: pointer;
  background: #409EFF;
  border: 1px solid #dcdfe6;
  color: #ffffff;
  -webkit-appearance: none;
  text-align: center;
  box-sizing: border-box;
  outline: none;
  margin: 10px;
  transition: .1s;
  font-weight: 500;
  /* 调整为mini尺寸的大小 */
  padding: 7px 15px;
  /* mini尺寸相对于普通尺寸有更小的内边距 */
  font-size: 12px;
  /* Element UI中mini尺寸的字体大小 */
  border-radius: 4px;
}

.custom-el-button:hover {
  background: #66b1ff;
}

.custom-el-button:active {
  background: #3a8ee6;
}
</style>
