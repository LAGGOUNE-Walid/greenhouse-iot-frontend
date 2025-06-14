<template>

</template>

<script setup>
import { ref, computed } from "vue";
import { useRoute } from "vue-router";

import {
  GridIcon,
  CalenderIcon,
  UserCircleIcon,
  ChatIcon,
  MailIcon,
  DocsIcon,
  PieChartIcon,
  ChevronDownIcon,
  HorizontalDots,
  PageIcon,
  TableIcon,
  ListIcon,
  PlugInIcon,
} from "../../icons";
import SidebarWidget from "./SidebarWidget.vue";
import BoxCubeIcon from "@/icons/BoxCubeIcon.vue";
import { useSidebar } from "@/composables/useSidebar";

const route = useRoute();

const { isExpanded, isMobileOpen, isHovered, openSubmenu } = useSidebar();

const menuGroups = [
  {
    title: "Menu",
    items: [
      {
        icon: GridIcon,
        name: "Dashboard",
        path: "/",
      },
      // ... Add other menu items here
    ],
  },
];

const isActive = (path) => route.path === path;

const toggleSubmenu = (groupIndex, itemIndex) => {
  const key = `${groupIndex}-${itemIndex}`;
  openSubmenu.value = openSubmenu.value === key ? null : key;
};

const isAnySubmenuRouteActive = computed(() => {
  return menuGroups.some((group) =>
    group.items.some(
      (item) =>
        item.subItems && item.subItems.some((subItem) => isActive(subItem.path))
    )
  );
});

const isSubmenuOpen = (groupIndex, itemIndex) => {
  const key = `${groupIndex}-${itemIndex}`;
  return (
    openSubmenu.value === key ||
    (isAnySubmenuRouteActive.value &&
      menuGroups[groupIndex].items[itemIndex].subItems?.some((subItem) =>
        isActive(subItem.path)
      ))
  );
};

const startTransition = (el) => {
  el.style.height = "auto";
  const height = el.scrollHeight;
  el.style.height = "0px";
  el.offsetHeight; // force reflow
  el.style.height = height + "px";
};

const endTransition = (el) => {
  el.style.height = "";
};
</script>
