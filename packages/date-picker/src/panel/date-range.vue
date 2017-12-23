<template>
  <transition name="el-zoom-in-top" @after-leave="$emit('dodestroy')">
    <div
      v-show="visible"
      class="el-picker-panel el-date-range-picker el-popper"
      :class="[{
        'has-sidebar': $slots.sidebar || shortcuts,
        'has-time': showTime
      }, popperClass]">
      <div class="el-picker-panel__body-wrapper">
        <slot name="sidebar" class="el-picker-panel__sidebar"></slot>
        <div class="el-picker-panel__sidebar" v-if="shortcuts">
          <button
            type="button"
            class="el-picker-panel__shortcut"
            v-for="shortcut in shortcuts"
            @click="handleShortcutClick(shortcut)">{{shortcut.text}}</button>
        </div>
        <div class="el-picker-panel__body">
          <div class="el-date-range-picker__time-header" v-if="showTime">
            <span class="el-date-range-picker__editors-wrap">
              <span class="el-date-range-picker__time-picker-wrap">
                <el-input
                  size="small"
                  :disabled="rangeState.selecting"
                  ref="minInput"
                  :placeholder="t('el.datepicker.startDate')"
                  class="el-date-range-picker__editor"
                  :value="minVisibleDate"
                  @input.native="handleDateInput($event, 'min')"
                  @change.native="handleDateChange($event, 'min')" />
              </span>
              <span class="el-date-range-picker__time-picker-wrap">
                <el-input
                  size="small"
                  :disabled="rangeState.selecting"
                  :placeholder="t('el.datepicker.startTime')"
                  class="el-date-range-picker__editor"
                  :value="minVisibleTime"
                  @focus="minTimePickerVisible = !minTimePickerVisible"
                  @change.native="handleTimeChange($event, 'min')" />
                <time-picker
                  ref="minTimePicker"
                  @pick="handleMinTimePick"
                  :time-arrow-control="arrowControl"
                  :visible="minTimePickerVisible"
                  @mounted="$refs.minTimePicker.format=timeFormat">
                </time-picker>
              </span>
            </span>
            <span class="el-icon-arrow-right"></span>
            <span class="el-date-range-picker__editors-wrap is-right">
              <span class="el-date-range-picker__time-picker-wrap">
                <el-input
                  size="small"
                  :disabled="rangeState.selecting"
                  :placeholder="t('el.datepicker.endDate')"
                  class="el-date-range-picker__editor"
                  :value="maxVisibleDate"
                  :readonly="!minDate"
                  @input.native="handleDateInput($event, 'max')"
                  @change.native="handleDateChange($event, 'max')" />
              </span>
              <span class="el-date-range-picker__time-picker-wrap">
                <el-input
                  size="small"
                  :disabled="rangeState.selecting"
                  ref="maxInput"
                  :placeholder="t('el.datepicker.endTime')"
                  class="el-date-range-picker__editor"
                  :value="maxVisibleTime"
                  @focus="minDate && (maxTimePickerVisible = !maxTimePickerVisible)"
                  :readonly="!minDate"
                  @change.native="handleTimeChange($event, 'max')" />
                <time-picker
                  ref="maxTimePicker"
                  @pick="handleMaxTimePick"
                  :time-arrow-control="arrowControl"
                  :visible="maxTimePickerVisible"
                  @mounted="$refs.maxTimePicker.format=timeFormat">
                </time-picker>
              </span>
            </span>
          </div>
          <div class="el-custom-button-all" v-if="yearAll && yearAll.length > 0 || monthAll">
            <ul v-if="yearAll" class="el-custom-button-year-all">
              <li v-for="(n, index) in yearAll"
                :class="buttonClassYear(index)">
                <button
                  type="button"
                  class="el-picker-panel__icon-btn"
                  @click="clickYear(n, index)">{{n}}年</button>
              </li>
            </ul>
            <ul v-if="monthAll" class="el-custom-button-month-all">
              <li v-for="(n, index) in 12"
                :class="buttonClassMonth(index)">
                <button
                  type="button"
                  class="el-picker-panel__icon-btn"
                  @click="clickMonth(n)">{{n}}月</button>
              </li>
            </ul>
          </div>
          <div class="el-picker-panel__content el-date-range-picker__content is-left">
            <div v-if="!yearAll || !monthAll" class="el-date-range-picker__header">
              <button
                type="button"
                @click="leftPrevYear"
                class="el-picker-panel__icon-btn el-icon-d-arrow-left"></button>
              <button
                type="button"
                @click="leftPrevMonth"
                class="el-picker-panel__icon-btn el-icon-arrow-left"></button>
              <button
                type="button"
                @click="leftNextYear"
                v-if="unlinkPanels"
                :disabled="!enableYearArrow"
                :class="{ 'is-disabled': !enableYearArrow }"
                class="el-picker-panel__icon-btn el-icon-d-arrow-right"></button>
              <button
                type="button"
                @click="leftNextMonth"
                v-if="unlinkPanels"
                :disabled="!enableMonthArrow"
                :class="{ 'is-disabled': !enableMonthArrow }"
                class="el-picker-panel__icon-btn el-icon-arrow-right"></button>
              <div>{{ leftLabel }}</div>
            </div>
            <date-table
              selection-mode="range"
              :date="leftDate"
              :default-value="defaultValue"
              :min-date="minDate"
              :max-date="maxDate"
              :range-state="rangeState"
              :disabled-date="disabledDate"
              @changerange="handleChangeRange"
              :first-day-of-week="firstDayOfWeek"
              @mouseup.native="handleMouseUp"
              @mousedown.native="handleMouseDown"
              @pick="handleRangePick">
            </date-table>
          </div>
          <div class="el-picker-panel__content el-date-range-picker__content is-right">
            <div v-if="!yearAll || !monthAll" class="el-date-range-picker__header">
              <button
                type="button"
                @click="rightPrevYear"
                v-if="unlinkPanels"
                :disabled="!enableYearArrow"
                :class="{ 'is-disabled': !enableYearArrow }"
                class="el-picker-panel__icon-btn el-icon-d-arrow-left"></button>
              <button
                type="button"
                @click="rightPrevMonth"
                v-if="unlinkPanels"
                :disabled="!enableMonthArrow"
                :class="{ 'is-disabled': !enableMonthArrow }"
                class="el-picker-panel__icon-btn el-icon-arrow-left"></button>
              <button
                type="button"
                @click="rightNextYear"
                class="el-picker-panel__icon-btn el-icon-d-arrow-right"></button>
              <button
                type="button"
                @click="rightNextMonth"
                class="el-picker-panel__icon-btn el-icon-arrow-right"></button>
              <div>{{ rightLabel }}</div>
            </div>
            <date-table
              selection-mode="range"
              :date="rightDate"
              :default-value="defaultValue"
              :min-date="minDate"
              :max-date="maxDate"
              :range-state="rangeState"
              :disabled-date="disabledDate"
              @changerange="handleChangeRange"
              :first-day-of-week="firstDayOfWeek"
              @mouseup.native="handleMouseUp"
              @mousedown.native="handleMouseDown"
              @pick="handleRangePick">
            </date-table>
          </div>
        </div>
      </div>
      <div class="el-picker-panel__footer" v-if="showTime">
        <el-button
          size="mini"
          type="text"
          class="el-picker-panel__link-btn"
          @click="handleClear">
          {{ t('el.datepicker.clear') }}
        </el-button>
        <el-button
          plain
          size="mini"
          class="el-picker-panel__link-btn"
          :disabled="btnDisabled"
          @click="handleConfirm()">
          {{ t('el.datepicker.confirm') }}
        </el-button>
      </div>
    </div>
  </transition>
</template>

<script type="text/babel">
  import {
    formatDate,
    parseDate,
    isDate,
    modifyDate,
    modifyTime,
    prevMonth,
    nextMonth
  } from '../util';
  import Locale from 'element-ui/src/mixins/locale';
  import TimePicker from './time';
  import DateTable from '../basic/date-table';
  import ElInput from 'element-ui/packages/input';
  import ElButton from 'element-ui/packages/button';

  const advanceDate = (date, amount) => {
    return new Date(new Date(date).getTime() + amount);
  };

  const calcDefaultValue = (defaultValue) => {
    if (Array.isArray(defaultValue)) {
      return [new Date(defaultValue[0]), new Date(defaultValue[1])];
    } else if (defaultValue) {
      return [new Date(defaultValue), advanceDate(defaultValue, 24 * 60 * 60 * 1000)];
    } else {
      return [new Date(), advanceDate(Date.now(), 24 * 60 * 60 * 1000)];
    }
  };

  export default {
    mixins: [Locale],

    computed: {
      btnDisabled() {
        return !(this.minDate && this.maxDate && !this.selecting);
      },

      leftLabel() {
        return this.leftDate.getFullYear() + ' ' + this.t('el.datepicker.year') + ' ' + this.t(`el.datepicker.month${ this.leftDate.getMonth() + 1 }`);
      },

      rightLabel() {
        return this.rightDate.getFullYear() + ' ' + this.t('el.datepicker.year') + ' ' + this.t(`el.datepicker.month${ this.rightDate.getMonth() + 1 }`);
      },

      leftYear() {
        return this.leftDate.getFullYear();
      },

      leftMonth() {
        return this.leftDate.getMonth();
      },

      leftMonthDate() {
        return this.leftDate.getDate();
      },

      rightYear() {
        return this.rightDate.getFullYear();
      },

      rightMonth() {
        return this.rightDate.getMonth();
      },

      rightMonthDate() {
        return this.rightDate.getDate();
      },

      minVisibleDate() {
        return this.minDate ? formatDate(this.minDate) : '';
      },

      maxVisibleDate() {
        return (this.maxDate || this.minDate) ? formatDate(this.maxDate || this.minDate) : '';
      },

      minVisibleTime() {
        return this.minDate ? formatDate(this.minDate, 'HH:mm:ss') : '';
      },

      maxVisibleTime() {
        return (this.maxDate || this.minDate) ? formatDate(this.maxDate || this.minDate, 'HH:mm:ss') : '';
      },

      dateFormat() {
        if (this.format) {
          return this.format.replace('HH:mm', '').replace(':ss', '').trim();
        } else {
          return 'yyyy-MM-dd';
        }
      },

      timeFormat() {
        if (this.format && this.format.indexOf('ss') === -1) {
          return 'HH:mm';
        } else {
          return 'HH:mm:ss';
        }
      },

      enableMonthArrow() {
        const nextMonth = (this.leftMonth + 1) % 12;
        const yearOffset = this.leftMonth + 1 >= 12 ? 1 : 0;
        return this.unlinkPanels && new Date(`${this.leftYear + yearOffset}-${nextMonth + 1}`) < new Date(`${this.rightYear}-${this.rightMonth + 1}`);
      },

      enableYearArrow() {
        return this.unlinkPanels && this.rightYear * 12 + this.rightMonth - (this.leftYear * 12 + this.leftMonth + 1) >= 12;
      }
    },

    data() {
      return {
        popperClass: '',
        value: [],
        defaultValue: null,
        minDate: '',
        maxDate: '',
        leftDate: new Date(),
        rightDate: nextMonth(new Date()),
        rangeState: {
          endDate: null,
          selecting: false,
          row: null,
          column: null
        },
        showTime: false,
        shortcuts: '',
        visible: '',
        disabledDate: '',
        firstDayOfWeek: 7,
        minTimePickerVisible: false,
        maxTimePickerVisible: false,
        format: '',
        arrowControl: false,
        unlinkPanels: false,
        // 展开年份按钮
        yearAll: null,
        // 展开月份按钮
        monthAll: false,
        // 年月按钮样式选中记录
        buttonClass: {
          year: [0],
          month: [1, 2]
        }
      };
    },

    watch: {
      minDate(val) {
        this.$nextTick(() => {
          if (this.$refs.maxTimePicker && this.maxDate && this.maxDate < this.minDate) {
            const format = 'HH:mm:ss';
            this.$refs.maxTimePicker.selectableRange = [
              [
                parseDate(formatDate(this.minDate, format), format),
                parseDate('23:59:59', format)
              ]
            ];
          }
        });
        if (val && this.$refs.minTimePicker) {
          this.$refs.minTimePicker.date = val;
          this.$refs.minTimePicker.value = val;
        }
      },

      maxDate(val) {
        if (val && this.$refs.maxTimePicker) {
          this.$refs.maxTimePicker.date = val;
          this.$refs.maxTimePicker.value = val;
        }
      },

      minTimePickerVisible(val) {
        if (val) {
          this.$nextTick(() => {
            this.$refs.minTimePicker.date = this.minDate;
            this.$refs.minTimePicker.value = this.minDate;
            this.$refs.minTimePicker.adjustSpinners();
          });
        }
      },

      maxTimePickerVisible(val) {
        if (val) {
          this.$nextTick(() => {
            this.$refs.maxTimePicker.date = this.maxDate;
            this.$refs.maxTimePicker.value = this.maxDate;
            this.$refs.maxTimePicker.adjustSpinners();
          });
        }
      },

      value(newVal) {
        if (!newVal) {
          this.minDate = null;
          this.maxDate = null;
        } else if (Array.isArray(newVal)) {
          this.minDate = isDate(newVal[0]) ? new Date(newVal[0]) : null;
          this.maxDate = isDate(newVal[1]) ? new Date(newVal[1]) : null;
          // NOTE: currently, maxDate = minDate + 1 month
          //       should allow them to be set individually in the future
          if (this.minDate) {
            this.leftDate = this.minDate;
            this.rightDate = this.unlinkPanels && this.maxDate
              ? this.maxDate
              : nextMonth(this.leftDate);
          } else {
            this.leftDate = calcDefaultValue(this.defaultValue)[0];
            this.rightDate = nextMonth(this.leftDate);
          }
        }
      },

      defaultValue(val) {
        if (!Array.isArray(this.value)) {
          const [left, right] = calcDefaultValue(val);
          this.leftDate = left;
          this.rightDate = val && val[1] && this.unlinkPanels
            ? right
            : nextMonth(this.leftDate);
        }
        if (this.yearAll || this.monthAll) {
          let year = [
            this.leftYear,
            this.rightYear
          ];
          let month = [
            this.leftDate.getMonth() + 1,
            this.rightDate.getMonth() + 1
          ];
          if (this.yearAll) {
            this.yearAll.forEach((v, i) => {
              if (v === year[0]) year[0] = i;
              if (v === year[1]) {
                year[1] = i;
              }
            });
            // 默认时间不在年份列表内设为头年或尾年
            let length = this.yearAll.length - 1;
            if (this.yearAll[0] >= this.leftYear) {
              this.leftDate = modifyDate(this.leftDate, this.yearAll[0], 0, this.leftMonthDate);
              this.rightDate = modifyDate(this.rightDate, this.yearAll[0], 1, this.rightMonthDate);
              year = [0];
              month = [1, 2];
            } else if (this.yearAll[length] <= this.leftYear) {
              this.leftDate = modifyDate(this.leftDate, this.yearAll[length], 10, this.leftMonthDate);
              this.rightDate = modifyDate(this.rightDate, this.yearAll[length], 11, this.rightMonthDate);
              year = [length];
              month = [11, 12];
            } else if (year[0] === year[1]) year.pop();
          }
          this.$set(this, 'buttonClass', {year, month});
        }
      }
    },

    methods: {
      // 年份选中样式
      buttonClassYear(i) {
        if (this.buttonClass.year.some(v => v === i)) {
          let length = this.buttonClass.year.length;
          return length === 1 ? ['el-custom-button-active-only'] : ['el-custom-button-active'];
        }
      },
      // 年份选中样式
      buttonClassMonth(i) {
        i = i + 1;
        if (this.buttonClass.month.some(v => v === i)) {
          return this.buttonClass.month[1] === i && i === 1 ? ['el-custom-button-active-first'] : ['el-custom-button-active'];
        }
      },
      // 点击具体年份切换
      clickYear(v, i) {
        let arr;
        let yearAllLast = this.yearAll[this.yearAll.length - 1];
        if (this.buttonClass.month[0] === 12 && this.buttonClass.month[1] === 1 && this.yearAll[i] !== yearAllLast) {
          arr = [i, i + 1];
        } else arr = [i];
        this.$set(this.buttonClass, 'year', arr);
        if (this.yearAll[i] !== yearAllLast || this.buttonClass.month[0] !== 12) {
          this.leftDate = modifyDate(this.leftDate, v, this.leftMonth, this.leftMonthDate);
          this.rightDate = modifyDate(this.rightDate, v, this.rightMonth, this.rightMonthDate);
        } else this.clickMonth(11); // 如果是最后一年就跳到11月
      },
      // 点击具体月份
      clickMonth(v) {
        let arr;
        let yearArr;
        let yearLength = this.buttonClass.year.length;
        let rightYear;
        // 最后一年12月份跳11月
        if (v >= 12 && this.yearAll.length - 1 === this.buttonClass.year[0]) v = 11;
        if (v >= 12) { // 跨年
          arr = [12, 1];
          if (yearLength === 1) {
            yearArr = [
              this.buttonClass.year[0],
              this.buttonClass.year[0] + 1
            ];
          } else {
            yearArr = [
              this.buttonClass.year[1],
              this.buttonClass.year[0] + 1
            ];
          }
          rightYear = this.rightYear;
        } else {
          arr = [v, v + 1];
          if (yearLength !== 1) yearArr = [this.buttonClass.year[0]];
          rightYear = this.leftYear;
        }
        if (yearArr) this.$set(this.buttonClass, 'year', yearArr);
        this.$set(this.buttonClass, 'month', arr);
        this.leftDate = modifyDate(this.leftDate, this.leftYear, arr[0] - 1, this.leftMonthDate);
        this.rightDate = modifyDate(this.rightDate, rightYear, arr[1] - 1, this.rightMonthDate);
      },

      handleClear() {
        this.minDate = null;
        this.maxDate = null;
        this.leftDate = calcDefaultValue(this.defaultValue)[0];
        this.rightDate = nextMonth(this.leftDate);
        this.$emit('pick', null);
      },

      handleChangeRange(val) {
        this.minDate = val.minDate;
        this.maxDate = val.maxDate;
        this.rangeState = val.rangeState;
      },

      handleDateInput(event, type) {
        const value = event.target.value;
        if (value.length !== this.dateFormat.length) return;
        const parsedValue = parseDate(value, this.dateFormat);

        if (parsedValue) {
          if (typeof this.disabledDate === 'function' &&
            this.disabledDate(new Date(parsedValue))) {
            return;
          }
          if (type === 'min') {
            this.minDate = new Date(parsedValue);
            this.leftDate = new Date(parsedValue);
            this.rightDate = nextMonth(this.leftDate);
          } else {
            this.maxDate = new Date(parsedValue);
            this.leftDate = prevMonth(parsedValue);
            this.rightDate = new Date(parsedValue);
          }
        }
      },

      handleDateChange(event, type) {
        const value = event.target.value;
        const parsedValue = parseDate(value, this.dateFormat);
        if (parsedValue) {
          if (type === 'min') {
            this.minDate = modifyDate(this.minDate, parsedValue.getFullYear(), parsedValue.getMonth(), parsedValue.getDate());
            if (this.minDate > this.maxDate) {
              this.maxDate = this.minDate;
            }
          } else {
            this.maxDate = modifyDate(this.maxDate, parsedValue.getFullYear(), parsedValue.getMonth(), parsedValue.getDate());
            if (this.maxDate < this.minDate) {
              this.minDate = this.maxDate;
            }
          }
        }
      },

      handleTimeChange(event, type) {
        const value = event.target.value;
        const parsedValue = parseDate(value, this.timeFormat);
        if (parsedValue) {
          if (type === 'min') {
            this.minDate = modifyTime(this.minDate, parsedValue.getHours(), parsedValue.getMinutes(), parsedValue.getSeconds());
            if (this.minDate > this.maxDate) {
              this.maxDate = this.minDate;
            }
            this.$refs.minTimePicker.value = this.minDate;
            this.minTimePickerVisible = false;
          } else {
            this.maxDate = modifyTime(this.maxDate, parsedValue.getHours(), parsedValue.getMinutes(), parsedValue.getSeconds());
            if (this.maxDate < this.minDate) {
              this.minDate = this.maxDate;
            }
            this.$refs.maxTimePicker.value = this.minDate;
            this.maxTimePickerVisible = false;
          }
        }
      },

      // 鼠标按下触发事件
      handleMouseDown(event) {
        this.mousedownPick && this.mousedownPick.call(this, event);
      },
      // 鼠标弹起触发事件
      handleMouseUp(event) {
        this.mouseupPick && this.mouseupPick.call(this, event);
      },
      handleRangePick(val, close = true) {
        if (this.maxDate === val.maxDate && this.minDate === val.minDate) {
          return;
        }
        this.onPick && this.onPick(val);
        this.maxDate = val.maxDate;
        this.minDate = val.minDate;

        // workaround for https://github.com/ElemeFE/element/issues/7539, should remove this block when we don't have to care about Chromium 55 - 57
        setTimeout(() => {
          this.maxDate = val.maxDate;
          this.minDate = val.minDate;
        }, 10);
        if (!close || this.showTime) return;
        this.handleConfirm();
      },

      handleShortcutClick(shortcut) {
        if (shortcut.onClick) {
          shortcut.onClick(this);
        }
      },

      handleMinTimePick(value, visible, first) {
        this.minDate = this.minDate || new Date();
        if (value) {
          this.minDate = modifyTime(this.minDate, value.getHours(), value.getMinutes(), value.getSeconds());
        }

        if (!first) {
          this.minTimePickerVisible = visible;
        }

        if (!this.maxDate || this.maxDate && this.maxDate.getTime() < this.minDate.getTime()) {
          this.maxDate = new Date(this.minDate);
        }
      },

      handleMaxTimePick(value, visible, first) {
        if (this.maxDate && value) {
          this.maxDate = modifyTime(this.maxDate, value.getHours(), value.getMinutes(), value.getSeconds());
        }

        if (!first) {
          this.maxTimePickerVisible = visible;
        }

        if (this.maxDate && this.minDate && this.minDate.getTime() > this.maxDate.getTime()) {
          this.minDate = new Date(this.maxDate);
        }
      },

      leftPrevYear() {
        this.leftDate = modifyDate(this.leftDate, this.leftYear - 1, this.leftMonth, this.leftMonthDate);
        if (!this.unlinkPanels) {
          this.rightDate = nextMonth(this.leftDate);
        }
      },

      leftNextYear() {
        this.leftDate = modifyDate(this.leftDate, this.leftYear + 1, this.leftMonth, this.leftMonthDate);
      },

      leftPrevMonth() {
        this.leftDate = prevMonth(this.leftDate);
        if (!this.unlinkPanels) {
          this.rightDate = nextMonth(this.leftDate);
        }
      },

      leftNextMonth() {
        this.leftDate = nextMonth(this.leftDate);
      },

      rightPrevYear() {
        this.rightDate = modifyDate(this.rightDate, this.rightYear - 1, this.rightMonth, this.rightMonthDate);
      },

      rightNextYear() {
        if (!this.unlinkPanels) {
          this.leftDate = modifyDate(this.leftDate, this.leftYear + 1, this.leftMonth, this.leftMonthDate);
          this.rightDate = nextMonth(this.leftDate);
        } else {
          this.rightDate = modifyDate(this.rightDate, this.rightYear + 1, this.rightMonth, this.rightMonthDate);
        }
      },

      rightPrevMonth() {
        this.rightDate = prevMonth(this.rightDate);
      },

      rightNextMonth() {
        if (!this.unlinkPanels) {
          this.leftDate = nextMonth(this.leftDate);
          this.rightDate = nextMonth(this.leftDate);
        } else {
          this.rightDate = nextMonth(this.rightDate);
        }
      },

      // 修改日期: 最后一个参数为要修改的日期面板字段
      modifyDate() {
        this[arguments[arguments.length - 1]] = modifyDate(...arguments);
      },

      handleConfirm(visible = false) {
        this.$emit('pick', [this.minDate, this.maxDate], visible);
      },

      isValidValue(value) {
        return Array.isArray(value) &&
          value && value[0] && value[1] &&
          isDate(value[0]) && isDate(value[1]) &&
          value[0].getTime() <= value[1].getTime() && (
            typeof this.disabledDate === 'function'
            ? !this.disabledDate(value[0]) && !this.disabledDate(value[1])
            : true
          );
      }
    },

    components: { TimePicker, DateTable, ElInput, ElButton }
  };
</script>
