<template>
    <div class="input-box no-autofill-style">
        <div class="input-content"
            @keydown="keydown"
            @keyup="keyup"
            @mousewheel="mousewheel"
            @input="inputEvent">
            <template v-for="(v, index) in input" >
                <template v-if="index == 0">
                    <input
                        :key="index"
                        @paste="paste"
                        max="9"
                        min="0"
                        maxlength="1"
                        :data-index="index"
                        v-model.trim.number="input[index]"
                        ref="firstinput"
                        type="number"/>
                </template>
                <template v-else>
                    <input
                        :key="index"
                        @paste="paste"
                        max="9"
                        min="0"
                        maxlength="1"
                        :data-index="index"
                        v-model.trim.number="input[index]"
                        type="number"/>
                </template>
            </template>
        </div>
    </div>
</template>

<script>
/**
 * 返回组合后的value
 * 当且仅当value全部有值时返回
 *
 * 根据len判断长度
 *
 * 根据type生产class生成样式，
 *
 */
    export default {
        props: {
            value: {
                type: [String, Number]
            },
            len: {
                type: Number,
                default: 6
            },
            code: {
                type: [String, Number]
            }
        },
        data() {
            return {
                pasteResult: [],
                lastEventIndex: -1,
                inputEventValues: [],
            };
        },
        computed: {
            input() {
                var len = parseInt(this.len)
                if(len <= 1) {
                    len = 1
                }
                return new Array(len);
                /*
                var reg = "^\\d{" + this.len + "}$";
                var r = new RegExp(reg, 'g');

                if (this.code && Array.isArray(this.code) && this.code.length === this.len) {
                    return this.code;
                } else if (this.code && r.test(this.code.toString())) {
                    return this.code.toString().split('');
                } else if (this.pasteResult.length === this.len) {
                    return this.pasteResult;
                } else {
                    return new Array(this.len);
                } */
            },

        },
        watch: {
            input() {
                this.valueChange();
            },
        },
        mounted() {
            // 等待dom渲染完成，在执行focus,否则无法获取到焦点
            this.$nextTick(() => {
                this.$refs['firstinput'] && this.$refs['firstinput'].focus && this.$refs['firstinput'].focus();
            })
        },
        methods: {
            // 解决一个输入框输入多个字符
            inputEvent(e) {
                var index = e.target.dataset.index * 1;
                var el = e.target;
                // 假设验证码是5418
                // 在iOS上使用自动填充密码时，不会触发paste事件，
                // keyup和input event都在input中会触发四次，el.value 分别是 "5", "54", "51", "58"
                if(index == this.lastEventIndex) {
                  let val = el.value;
                  this.inputEventValues = this.inputEventValues.concat(val.length>1?val.substring(1,2):val);
                  if(this.inputEventValues.length == this.len) {
                    // console.log(this.inputEventValues); // ["5", "4", "1", "8"]
                    document.activeElement.blur();
                    this.inputEventValues.forEach((t,index)=>{
                      this.$set(this.input, index, t);
                    });
                    this.$emit('complete', this.input);
                    this.inputEventValues = [];
                    return;
                  }
                } else {
                  this.lastEventIndex = index;
                  this.inputEventValues = [el.value];
                  // console.log(this.inputEventValues);
                }
                // console.log('inputEvent',el.value);
                // console.log('inputEvent',e.code);
                // console.log('inputEvent',e.which);
                el.value = el.value.replace(/Digit|Numpad/i, '').replace(/1/g, '').slice(0, 1);
                //
                this.$set(this.input, index, el.value)
                this.$nextTick(function() {
                    this.valueChange()
                })

            },
            keydown(e) {
                var index = e.target.dataset.index * 1;
                var el = e.target;
                if (e.key === 'Backspace') {
                    if (this.input[index].length > 0) {
                        this.$set(this.input, index, '')
                    } else {
                        if (el.previousElementSibling) {
                            try {
                                el.previousElementSibling.focus()
                            } catch(e) {
                                e
                            }
                            this.$set(this.input, index - 1, '')
                        }
                    }
                } else if (e.key === 'Delete') {
                    if (this.input[index].length > 0) {
                        this.$set(this.input, index, '')
                    } else {
                        if (el.nextElementSibling) {
                            this.$set(this.input, index = 1, '')
                        }
                    }
                    if (el.nextElementSibling) {
                        el.nextElementSibling.focus()
                    }
                } else if (e.key === 'Home') {
                    el.parentElement.children[0] && el.parentElement.children[0].focus()
                } else if (e.key === 'End') {
                    el.parentElement.children[this.input.length - 1] && el.parentElement.children[this.input.length - 1].focus()
                } else if (e.key === 'ArrowLeft') {
                    if (el.previousElementSibling) {
                        el.previousElementSibling.focus()
                    }
                } else if (e.key === 'ArrowRight') {
                    if (el.nextElementSibling) {
                        el.nextElementSibling.focus()
                    }
                } else if (e.key === 'ArrowUp') {
                    if (this.input[index] * 1 < 9) {
                        this.$set(this.input, index, (this.input[index] * 1 + 1).toString());
                    }
                } else if (e.key === 'ArrowDown') {
                    if (this.input[index] * 1 > 0) {
                        this.$set(this.input, index, (this.input[index] * 1 - 1).toString());
                    }
                }
            },
            keyup(e) {
                var index = e.target.dataset.index * 1;
                var el = e.target;
                // console.log('keyup',el.value);
                // console.log('keyup',e.code);
                // console.log('keyup',e.which);

                // android 上 e.code 一直为空，用e.which来代替e.code
                // https://stackoverflow.com/questions/17139039/keycode-is-always-zero-in-chrome-for-android
                const isNumber = e.which >= 48 && e.which <= 57;

                // 解决输入e的问题
                el.value = el.value.replace(/Digit|Numpad/i, '').replace(/1/g, '').slice(0, 1);
                if (/Digit|Numpad/i.test(e.code)||isNumber) {
                    // 必须在这里符直，否则输入框会是空值
                    if (/Digit|Numpad/i.test(e.code)) {
                      this.$set(this.input, index, e.code.replace(/Digit|Numpad/i, ''));
                    } else {
                      // android 上 e.code 一直为空，用e.which来代替e.code
                      this.$set(this.input, index, (e.which - 48).toString());
                    }
                    el.nextElementSibling && el.nextElementSibling.focus();
                    if (index === (this.len - 1)) {
                        document.activeElement.blur();
                        // el.blur()
                        this.$emit('complete', this.input);
                        // console.log('complete', this.input);
                        /* if (this.input.length === this.len) {
                            document.activeElement.blur();
                            el.blur()
                            this.$emit('complete', this.input);
                        } */
                    }
                } else {
                    if (this.input[index] === '') {
                        this.$set(this.input, index, '');
                    }
                }
                this.valueChange()
            },
            mousewheel(e) {
                var index = e.target.dataset.index;
                if (e.wheelDelta > 0) {
                    if (this.input[index] * 1 < 9) {
                        this.$set(this.input, index, (this.input[index] * 1 + 1).toString());
                    }
                } else if (e.wheelDelta < 0) {
                    if (this.input[index] * 1 > 0) {
                        this.$set(this.input, index, (this.input[index] * 1 - 1).toString());
                    }
                } else if (e.key === 'Enter') {
                    if (this.input.join('').length === this.len) {
                        document.activeElement.blur();
                        this.$emit('complete', this.input);
                    }
                }
            },
            paste(e) {
                // console.log(e);
                // 当进行粘贴时
                e.clipboardData.items[0].getAsString(str => {
                    // console.log(str);
                    if (str.toString().length === this.len) {
                        this.pasteResult = str.split('');
                        document.activeElement.blur();
                        this.pasteResult.forEach((t,index)=>{
                          this.$set(this.input, index, t);
                        });
                        this.$emit('complete', this.input);
                        this.pasteResult = [];
                    } else {
                        // 如果粘贴内容不合规，清除所有内容
                        this.input.forEach((t,index)=>{
                          this.$set(this.input, index, '');
                        });
                        document.activeElement.blur();
                    }
                })
                this.valueChange()
            },
            valueChange() {
                // console.log(this.input)
                var val = null;
                var state = true;
                this.input && this.input.forEach(d => {
                    if(d!== null && /^[0-9]{1}$/.test(d)) {

                    } else {
                        state = false;
                    }
                })
                if(state) {
                    val = this.input.join('')
                }
                this.$emit('input', val)
                this.$emit('change', val)
                // console.log(val)
            }
        },
    }
</script>

<style>
  div.no-autofill-style input:-webkit-autofill,
  div.no-autofill-style input:-webkit-autofill:hover,
  div.no-autofill-style input:-webkit-autofill:focus {
    transition: background-color 5000s ease-in-out 0s;
    -webkit-text-fill-color: #0c0c0c;
  }
</style>

<style scoped lang="scss">
    .input-box {
        .input-content {
            width: 100%;
            height: 60px;
            display: flex;
            align-items: center;
            justify-content: center;

            input {
                color: inherit;
                font-family: inherit;
                border: 0;
                outline: 0;
                border: 1px solid #919191;
                height: 60px;
                width: 60px;
                font-size: 40px;
                text-align: center;
                margin-left: 20px;
                margin-right: 20px;
                padding: 0;
            }
        }

        input::-webkit-outer-spin-button,
        input::-webkit-inner-spin-button {
            appearance: none;
            margin: 0;
        }
    }
</style>
