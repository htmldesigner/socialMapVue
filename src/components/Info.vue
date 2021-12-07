<template>
  <div>
    <h4>Информация</h4>
    <ul>
      <li>Инвалиды: {{ invalid }}</li>
      <li>Многодетные: {{ manyChildren }}</li>
      <li>Пенсионеры: {{ older }}</li>
      <li>Потеря кормильца: {{ lostParent }}</li>
      <li>Итого: {{ invalid + lostParent + manyChildren + older }}</li>
    </ul>
  </div>
</template>

<script>
export default {
  name: "Info",
  computed: {
    invalid() {
      return this.selected.filter(unit => unit.unitType === "invalid").length
    },
    lostParent() {
      return this.selected.filter(unit => unit.unitType === "lostparent").length
    },
    manyChildren() {
      return this.selected.filter(unit => unit.unitType === "manychildren").length
    },
    older() {
      return this.selected.filter(unit => unit.unitType === "older").length
    }
  },
  props: {
    selected: {
      type: Array,
      default: 0
    }
  }
}
</script>

<style scoped>
h4 {
  margin: 0 0 5px;
  color: #777;
  text-align: center;
}

ul {
  padding: 0;
}

ul li {
  list-style: none;
  margin: 6px;
}
</style>