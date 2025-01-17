<script>
import { onMounted, ref, inject, nextTick, markRaw } from "vue";
import {
  propsBinder,
  remapEvents,
  WINDOW_OR_GLOBAL,
  GLOBAL_LEAFLET_OPT,
} from "../utils.js";
import { tooltipProps, setupTooltip } from "../functions/tooltip";
import { render } from "../functions/popper";

/**
 * Display a tooltip on the map
 */
export default {
  name: "LTooltip",
  props: tooltipProps,
  setup(props, context) {
    const leafletObject = ref({});
    const root = ref(null);

    const useGlobalLeaflet = inject(GLOBAL_LEAFLET_OPT);
    const bindTooltip = inject("bindTooltip");

    const { options, methods } = setupTooltip(props, leafletObject, context);

    onMounted(async () => {
      const { tooltip, DomEvent } = useGlobalLeaflet
        ? WINDOW_OR_GLOBAL.L
        : await import("leaflet/dist/leaflet-src.esm");

      leafletObject.value = markRaw(tooltip(options));

      propsBinder(methods, leafletObject.value, props);
      const listeners = remapEvents(context.attrs);
      DomEvent.on(leafletObject.value, listeners);
      leafletObject.value.setContent(props.content || root.value);
      bindTooltip({ leafletObject: leafletObject.value });
      nextTick(() => context.emit("ready", leafletObject.value));
    });

    return { root, leafletObject };
  },
  render() {
    return render(this.$slots);
  },
};
</script>
