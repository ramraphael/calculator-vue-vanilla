<template>
  <main class="calculator">
    <window-header></window-header>
    <calculator-display :currentOperand="currentOperand" :currentResult="currentResult"></calculator-display>
    <calculator-input
      @click-clear="clickClear"
      @toggle-positive-negative="togglePositiveNegative"
      @click-number="clickNumber($event)"
      @click-decimal="clickDecimal"
      @click-operator="clickOperator($event)"
      @click-equals="clickEquals"
    ></calculator-input>
  </main>
</template>

<script>
import WindowHeader from './TitleBar/WindowHeader';
import CalculatorDisplay from './CalculatorUI/CalculatorDisplay';
import CalculatorInput from './CalculatorUI/CalculatorInput';
export default {
  name: 'Calculator',
  components: {
    WindowHeader,
    CalculatorDisplay,
    CalculatorInput
  },
  data: () => ({
    currentOperand: '',
    firstOperand: '',
    operator: '',
    currentResult: ''
  }),
  methods: {
    clickClear() {
      // Clear all properties
      this.currentOperand = '';
      this.firstOperand = '';
      this.currentResult = '';
      this.operator = '';
    },
    togglePositiveNegative() {
      // Only currentOperand can be toggled, remove or add '-' as first character
      this.checkIfContinuingOperation();
      this.currentOperand[0] === '-'
        ? (this.currentOperand = this.currentOperand.slice(1))
        : (this.currentOperand = '-'.concat(this.currentOperand));
    },
    clickNumber(selectedNumber) {
      // If result exists, reset it. Append selected number to current operand
      if (this.currentResult) {
        this.currentResult = '';
      }
      this.currentOperand += selectedNumber;
    },
    clickDecimal() {
      this.checkIfContinuingOperation();
      if (!this.currentOperand) {
        this.currentOperand = '0.';
      } else if (!this.currentOperand.includes('.')) {
        // Concat more readable than template literals or '+' for strings
        this.currentOperand = this.currentOperand.concat('.');
      }
    },
    clickOperator(selectedOperator) {
      // If both firstOperand and currentOperand exist, evaluate and use result as first operand
      // Similary, use currentResult as first operand if it's already been calculated
      // Coming from React, It feels weird to be directly mutating state...
      if (this.firstOperand && this.currentOperand) {
        this.firstOperand = this.evaluate();
        this.currentOperand = '';
      } else if (this.currentResult) {
        this.firstOperand = this.currentResult;
        this.currentResult = '';
        this.currentOperand = '';
      } else {
        this.firstOperand = this.currentOperand;
        this.currentOperand = '';
      }
      this.operator = selectedOperator;
    },
    clickEquals() {
      // Only do calculations if there are no current results, and first and current operands exist
      if (this.firstOperand && this.currentOperand) {
        this.currentResult = this.evaluate();
        this.firstOperand = '';
        this.currentOperand = '';
      }
    },
    checkIfContinuingOperation() {
      if (!this.currentOperand && this.currentResult) {
        this.currentOperand = this.currentResult;
        this.currentResult = '';
      }
    },
    evaluate() {
      // Returns evaluation of firstOperand, the operator and currentOperand
      if (this.firstOperand && this.currentOperand && this.operator) {
        switch (this.operator) {
          case '+':
            return String(Number(this.firstOperand) + Number(this.currentOperand));
          case '-':
            return String(Number(this.firstOperand) - Number(this.currentOperand));
          case '%':
            return String((Number(this.firstOperand) / 100) * this.currentOperand);
          case '*':
            return String(Number(this.firstOperand) * Number(this.currentOperand));
          case '/':
            return String(Number(this.firstOperand) / Number(this.currentOperand));
          default:
            return console.log('Unknown operator');
        }
      }
    },
    onKeypress(e) {
      // keydown event listener
      const { key } = e;
      if (key.length === 1 && /[0-9]/.test(key)) {
        this.clickNumber(key);
      } else if (key === '.') {
        this.clickDecimalPoint();
      } else if (/^(\+|-|\*|\/|%)$/.test(key)) {
        this.clickOperator(key);
      } else if (key === '=' || key === 'Enter') {
        this.clickEquals();
      } else if (key === 'Escape') {
        this.clickClear();
      }
    }
  },
  mounted() {
    // Add event listeners for common keyboard shortcuts
    window.addEventListener('keydown', this.onKeypress);
  },
  beforeDestroy() {
    // Remove event listener in case of unmount to prevent memory leaks
    window.removeEventListener('keydown', this.onKeypress);
  }
};
</script>

<style lang="scss">
.calculator {
  background-color: #373737;
  border: 3px solid #373737;
  border-radius: 2.5%;
  box-shadow: rgba(55, 55, 55, 0.4) 2px 2px 5px 4px;
  display: flex;
  flex-direction: column;
  height: 45rem;
  width: 30rem;
}
</style>
