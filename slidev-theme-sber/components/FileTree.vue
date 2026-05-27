<script setup lang="ts">
import { ref, watch } from 'vue';

interface TreeNode {
  name: string;
  isOpen?: boolean;
  children?: TreeNode[];
  isDirectory?: boolean;
}

const props = withDefaults(defineProps<{
  items?: TreeNode[];
  expandAll?: boolean;
}>(), {
  expandAll: false,
  items: () => [
    {
      name: 'eslint',
      isOpen: false,
      isDirectory: true,
      children: [
        { name: '.git', isDirectory: true },
        { name: 'src', isDirectory: true },
        { name: 'package.json', isDirectory: false },
      ],
    },
    {
      name: 'ui-kit',
      isOpen: false,
      isDirectory: true,
      children: [
        { name: '.git', isDirectory: true },
        { name: 'src', isDirectory: true },
        { name: 'package.json', isDirectory: false },
      ],
    },
    {
      name: 'shared',
      isOpen: false,
      isDirectory: true,
      children: [
        { name: '.git', isDirectory: true },
        { name: 'src', isDirectory: true },
        { name: 'package.json', isDirectory: false },
      ],
    },
    {
      name: 'app-1',
      isOpen: false,
      isDirectory: true,
      children: [
        { name: '.git', isDirectory: true },
        { name: 'src', isDirectory: true },
        { name: 'package.json', isDirectory: false },
      ],
    },
    {
      name: 'app-2',
      isOpen: false,
      isDirectory: true,
      children: [
        { name: '.git', isDirectory: true },
        { name: 'src', isDirectory: true },
        { name: 'package.json', isDirectory: false },
      ],
    },
  ],
});

const treeData = ref<TreeNode[]>(
  props.items.map((item) => ({ ...item, isOpen: item.isOpen ?? false }))
);

watch(
  () => props.items,
  (newItems) => {
    treeData.value = newItems.map((item) => ({ ...item, isOpen: item.isOpen ?? false }));
  },
);

watch(
  () => props.expandAll,
  (shouldExpand) => {
    treeData.value.forEach((node) => {
      node.isOpen = shouldExpand;
    });
  },
);

const toggleNode = (node: TreeNode) => {
  if (node.children) {
    node.isOpen = !node.isOpen;
  }
};

const getIcon = (node: TreeNode): string => {
  if (node.isDirectory === false) return '📄';
  return '📁';
};
</script>

<template>
  <div class="file-tree">
    <div class="tree-container">
      <div v-for="node in treeData" :key="node.name" class="tree-node">
        <div class="node-header" @click="toggleNode(node)">
          <span
            v-if="node.children"
            class="toggle-icon"
            :class="{ open: node.isOpen }"
          >
            ▶
          </span>
          <span v-else class="toggle-icon disabled">·</span>
          <span class="node-icon">{{ getIcon(node) }}</span>
          <span class="node-name">{{ node.name }}</span>
        </div>
        <transition name="expand">
          <div v-show="node.isOpen" class="children">
            <div
              v-for="child in node.children"
              :key="child.name"
              class="child-node"
            >
              <span class="node-icon">{{ getIcon(child) }}</span>
              <span class="node-name">{{ child.name }}</span>
            </div>
          </div>
        </transition>
      </div>
    </div>
  </div>
</template>

<style scoped>
.file-tree {
  display: flex;
  justify-content: center;
  align-items: flex-start;
  width: 100%;
  height: 100%;
  padding: 1rem 2rem;
  font-family: 'Courier New', monospace;
  overflow: hidden;
}

.tree-container {
  background: rgba(255, 255, 255, 0.05);
  border: 2px solid rgba(255, 255, 255, 0.1);
  border-radius: 12px;
  padding: 2rem;
  max-width: 600px;
  width: 100%;
  max-height: 70vh;
  overflow-y: auto;
  backdrop-filter: blur(10px);
}

.tree-node {
  margin-bottom: 0.5rem;
}

.node-header {
  display: flex;
  align-items: center;
  cursor: pointer;
  padding: 0.5rem;
  border-radius: 6px;
  transition: background-color 0.2s ease;
  user-select: none;
}

.node-header:hover {
  background-color: rgba(255, 255, 255, 0.1);
}

.toggle-icon {
  display: inline-block;
  width: 20px;
  margin-right: 0.5rem;
  text-align: center;
  transition: transform 0.3s ease;
  color: rgba(255, 255, 255, 0.7);
  font-size: 0.8rem;
}

.toggle-icon.open {
  transform: rotate(90deg);
}

.toggle-icon.disabled {
  opacity: 0.3;
  cursor: default;
}

.node-icon {
  margin-right: 0.5rem;
  font-size: 1.2rem;
}

.node-name {
  font-size: 1rem;
  color: rgba(255, 255, 255, 0.9);
  font-weight: 500;
}

.children {
  margin-left: 1.5rem;
  padding-left: 1rem;
  border-left: 2px solid rgba(255, 255, 255, 0.2);
  margin-top: 0.25rem;
}

.child-node {
  display: flex;
  align-items: center;
  padding: 0.35rem 0.5rem;
  border-radius: 6px;
  margin-bottom: 0.25rem;
  color: rgba(255, 255, 255, 0.8);
  font-size: 0.95rem;
  transition: background-color 0.2s ease;
}

.child-node:hover {
  background-color: rgba(255, 255, 255, 0.05);
}

/* Анимация раскрытия */
.expand-enter-active,
.expand-leave-active {
  transition: all 0.4s ease;
  overflow: hidden;
}

.expand-enter-from {
  opacity: 0;
  max-height: 0;
  transform: translateY(-10px);
}

.expand-leave-to {
  opacity: 0;
  max-height: 0;
  transform: translateY(-10px);
}

.expand-enter-to,
.expand-leave-from {
  opacity: 1;
  max-height: 500px;
  transform: translateY(0);
}

/* Темная тема */
:deep(.dark) {
  .tree-container {
    background: rgba(0, 0, 0, 0.3);
    border-color: rgba(255, 255, 255, 0.15);
  }
}
</style>
