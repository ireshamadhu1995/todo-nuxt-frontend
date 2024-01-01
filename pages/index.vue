<template>
    <div>
           <TheHeader />
    </div>
</template>

<script>
export default {
  created() {
    this.$router.push('/todos');
  }
};
</script>