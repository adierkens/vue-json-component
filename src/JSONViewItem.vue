<template>
  <div class="json-view-item">
    <!-- Handle Objects and Arrays-->
    <div v-if="data.type === 'object' || data.type === 'array'">
      <span @click.stop="toggleOpen" :class="classes" :style="keyColor">
        {{ data.key }}:
        <span class="properties">&nbsp;{{ lengthString }}</span>
      </span>
      <json-view-item
        v-for="child in data.children"
        :key="getKey(child)"
        :data="child"
        v-show="open"
        :maxDepth="maxDepth"
        :styles="styles"
      />
    </div>
    <!-- Handle Leaf Values -->
    <div v-if="data.type === 'value'">
      <span class="value-key" :style="valueKeyColor"> {{ data.key }}: </span>
      <span :style="getValueStyle(data.value)">
        {{ JSON.stringify(data.value) }}
      </span>
    </div>
  </div>
</template>

<script lang="ts">
import Vue from "vue";

export default Vue.extend({
  name: "json-view-item",
  data: function() {
    return {
      open: this.data.depth < this.maxDepth
    };
  },
  props: {
    data: {
      required: true,
      type: Object
    },
    maxDepth: {
      type: Number,
      required: false,
      default: 1
    },
    styles: {
      type: Object,
      required: true
    }
  },
  methods: {
    toggleOpen: function(): void {
      this.open = !this.open;
    },
    getKey: function(value: any): string {
      if (!isNaN(value.key)) {
        return value.key + ":";
      } else {
        return '"' + value.key + '":';
      }
    },
    getValueStyle: function(value: any): object {
      const type = typeof value;
      switch (type) {
        case "string":
          return { color: this.styles.string };
        case "number":
          return { color: this.styles.number };
        case "boolean":
          return { color: this.styles.boolean };
        case "object":
          return { color: this.styles.null };
        default:
          return { color: this.styles.valueKeyColor };
      }
    }
  },
  computed: {
    classes: function(): object {
      return {
        "data-key": true,
        opened: this.open
      };
    },
    lengthString: function(): string {
      return this.data.length === 1
        ? this.data.length + " property"
        : this.data.length + " properties";
    },
    keyColor: function(): object {
      return { color: this.styles.key };
    },
    valueKeyColor: function(): object {
      return { color: this.styles.valueKey };
    }
  }
});
</script>

<style lang="scss" scoped>
.json-view-item {
  margin-left: 20px;
  padding: 5px;
}

.value-key {
  font-weight: 600;
  margin-left: 15px;
}

.data-key {
  display: flex;
  align-items: center;
  border-radius: 2px;
  font-weight: 600;
  cursor: pointer;

  &:hover {
    background-color: rgba(0, 0, 0, 0.08);
  }

  .properties {
    font-weight: 300;
    opacity: 0.6;
    user-select: none;
  }
}

.data-key::before {
  color: #444;
  position: relative;
  left: 15px;
  margin-right: 25px;
  user-select: none;
  content: "\25b6";
  margin-top: -2px;
  margin-bottom: 2px;
}

.data-key.opened::before {
  margin-top: 0px;
  margin-bottom: 0px;
  transform: rotate(90deg);
}
</style>