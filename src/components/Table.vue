<template>
  <table>
    <slot></slot>
  </table>
</template>
<style>
table {
  border-collapse: collapse;
  width: 100%;
}

td,
th {
  text-align: left;
  padding: 0.5rem;
}

th {
  font-weight: bold;
}

tr {
  border-bottom: solid 1px rgb(225, 225, 225);
  transition: all 0.5s ease;
}

tr:last-child {
  border-bottom: solid 1px transparent;
}
</style>
