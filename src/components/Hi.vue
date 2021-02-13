<template>
  <p v-if="show">Hi</p>
</template>
<script>
export default {
  name: "Hi",
  props: {
    show: Boolean,
  },
};
</script>
<style>
#Hi {
  font-family: Helvetica, Arial, sans-serif;
  text-align: center;
}
</style>
