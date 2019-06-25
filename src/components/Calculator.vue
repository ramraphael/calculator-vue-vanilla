<template>
  <main class="calculator">
    <window-header></window-header>
    <calculator-display :currentOperand="currentOperand" :currentResult="currentResult"></calculator-display>
    <calculator-input
      @click-clear="clearAll"
      @toggle-positive-negative="togglePositiveNegative"
      @click-number="clickNumber($event)"
      @click-decimal="addDecimal"
      @click-operator="clickOperator($event)"
      @click-equals="evaluateResult"
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
    // Clears all properties
    clearAll() {
      this.currentOperand = '';
      this.firstOperand = '';
      this.currentResult = '';
      this.operator = '';
    },
    // Toggles leading '-' on currentOperand
    // If currentResult exists, currentOperand set to currentResult and currentResult is reset
    togglePositiveNegative() {
      this.checkIfContinuingOperation();
      this.currentOperand[0] === '-'
        ? (this.currentOperand = this.currentOperand.slice(1))
        : (this.currentOperand = '-'.concat(this.currentOperand));
    },
    // Resets currentResult if it exists
    // Otherwise appends selectedNumber to currentOperand
    clickNumber(selectedNumber) {
      if (this.currentResult) {
        this.currentResult = '';
      }
      this.currentOperand += selectedNumber;
    },
    // Checks if continuing operation, then add decimal point if it doesn't exist
    addDecimal() {
      this.checkIfContinuingOperation();
      if (!this.currentOperand) {
        this.currentOperand = '0.';
      } else if (!this.currentOperand.includes('.')) {
        // Concat more readable than template literals or '+' for strings
        this.currentOperand = this.currentOperand.concat('.');
      }
    },
    // Updates operator
    // If currentResult exists, uses it as first operand
    // Otherwise sets firstOperand to currentOperand
    // Resets currentOperand
    clickOperator(selectedOperator) {
      // Coming from React, It feels weird to be directly mutating state...
      evaluateResult();
      if (this.currentResult) {
        this.firstOperand = this.currentResult;
        this.currentResult = '';
        this.currentOperand = '';
      } else {
        this.firstOperand = this.currentOperand;
        this.currentOperand = '';
      }
      this.operator = selectedOperator;
    },
    // Sets currentResult and resets first and current operands
    evaluateResult() {
      if (this.firstOperand && this.currentOperand) {
        this.currentResult = this.checkResult();
        this.firstOperand = '';
        this.currentOperand = '';
      }
    },
    // Checks for continuing operations
    // Sets currentOperand to currentResult and resets currentResult
    checkIfContinuingOperation() {
      if (!this.currentOperand && this.currentResult) {
        this.currentOperand = this.currentResult;
        this.currentResult = '';
      }
    },
    // Returns evaluation of firstOperand, the operator and currentOperand
    checkResult() {
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
    // Keydown event handler
    onKeypress(e) {
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
