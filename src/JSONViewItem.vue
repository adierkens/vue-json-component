<template>
  <div class="json-view-item">
    <!-- Handle Objects and Arrays-->
    <div v-if="data.type === 'object' || data.type === 'array'">
      <div @click.stop="toggleOpen" class="data-key" :style="keyColor">
        <div :class="classes" :style="arrowStyles"></div>
        {{ data.key }}:
        <span class="properties">&nbsp;{{ lengthString }}</span>
      </div>
      <json-view-item
        v-on:selected="bubbleSelected"
        v-for="child in data.children"
        :key="getKey(child)"
        :data="child"
        v-show="open"
        :maxDepth="maxDepth"
        :styles="styles"
        :canSelect="canSelect"
      />
    </div>
    <!-- Handle Leaf Values -->
    <div
      :class="valueClasses"
      v-on:click="clickEvent(data)"
      v-if="data.type === 'value'"
    >
      <span :style="valueKeyColor"> {{ data.key }}: </span>
      <span :style="getValueStyle(data.value)">
        {{ JSON.stringify(data.value) }}
      </span>
    </div>
  </div>
</template>

<script lang="ts">
import Vue, { VueConstructor } from "vue";

export interface SelectedData {
  key: string;
  value: string;
  path: string;
}

export interface Data {
  [key: string]: string;
}

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
    },
    canSelect: {
      type: Boolean,
      required: false,
      default: false
    }
  },
  methods: {
    toggleOpen: function(): void {
      this.open = !this.open;
    },
    clickEvent: function(data: Data): void {
      this.$emit("selected", {
        key: data.key,
        value: data.value,
        path: data.path
      } as SelectedData);
    },
    bubbleSelected: function(data: Data): void {
      this.$emit("selected", data);
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
        "chevron-arrow": true,
        opened: this.open
      };
    },
    valueClasses: function(): object {
      return {
        "value-key": true,
        "can-select": this.canSelect
      };
    },
    arrowStyles: function(): object {
      return { width: this.styles.arrowSize, height: this.styles.arrowSize };
    },
    lengthString: function(): string {
      if (this.data.type === "array") {
        return this.data.length === 1
          ? this.data.length + " element"
          : this.data.length + " elements";
      }
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
}

.value-key {
  font-weight: 600;
  margin-left: 10px;
  border-radius: 2px;
  white-space: nowrap;
  padding: 5px 5px 5px 10px;

  &.can-select {
    cursor: pointer;
    &:hover {
      background-color: rgba(0, 0, 0, 0.08);
    }
  }
}

.data-key {
  display: flex;
  align-items: center;
  border-radius: 2px;
  font-weight: 600;
  cursor: pointer;
  white-space: nowrap;
  padding: 5px;

  &:hover {
    background-color: rgba(0, 0, 0, 0.08);
  }

  .properties {
    font-weight: 300;
    opacity: 0.6;
    user-select: none;
  }
}

.chevron-arrow {
  flex-shrink: 0;
  border-right: 4px solid #444;
  border-bottom: 4px solid #444;
  width: 6px;
  height: 6px;
  margin-right: 20px;
  margin-left: 5px;
  transform: rotate(-45deg);

  &.opened {
    margin-top: -3px;
    transform: rotate(45deg);
  }
}
</style>
