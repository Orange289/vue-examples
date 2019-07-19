<template>
  <a v-if="href" :href=href class="gozo-button gozo-button--link"><slot>Open&nbsp;account</slot></a>
  <button v-else :type="resultType" class="gozo-button"><slot>Stay&nbsp;in&nbsp;touch</slot></button>
</template>

<script>
export default {
  props: {
    href: String,
    type: String
  },
  computed: {
    resultType: function() {
      if (!this.type || this.type.toLowerCase() !== "submit") {
        return "button";
      } else if(this.type.toLowerCase() === "submit") {
        return "submit";
      }
    }
  }
}
</script>

<style lang="scss">
.gozo-button {
  @include gozo-button;

  &.loading {
    color: #fff;
    background-color: $color__red;
    background-image: linear-gradient(135deg,rgba(255,255,255,.15) 25%,transparent 25%,transparent 50%,rgba(255,255,255,.15) 50%,rgba(255,255,255,.15) 75%,transparent 75%,transparent);
    animation: ca-button-loading--big 2s linear infinite;
    background-size: 54px 54px;
    cursor: default;
  }

  &.gozo-button--form {
    @include gozo-button--form;
  }

  &.gozo-button--link {
    @include gozo-button--link;
  }

  &.gozo-button--main {
    @include gozo-button--main;
  }

  &.gozo-button--remain {
    @include gozo-button--remain;
  }

}

@keyframes ca-button-loading--big {
  from {
    background-position: 54px 0;
  }
  to {
    background-position: 0 0;
  }
}

</style>
