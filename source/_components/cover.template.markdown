---
layout: page
title: "Template Cover"
description: "Instructions how to integrate Template Cover into Home Assistant."
date: 2017-03-14 14:00
sidebar: true
comments: false
sharing: true
footer: true
ha_category: Cover
ha_release: 0.41
ha_iot_class: "Local Push"
logo: home-assistant.png
---

The `template` platform creates covers that combine components.

For example, if you have a window cover with a switch that operates the motor and a proximity sensor that allows you to know whether the window is open or closed, you can combine these into a cover that knows whether the window cover is open or closed.

This can simplify the gui, and make it easier to write automations. You can mark the components you have combined as `hidden` so they don't appear themselves.

To enable Template covers in your installation, add the following to your `configuration.yaml` file:

```yaml
# Example configuration.yaml entry
cover:
  - platform: template
    covers:
      living_room:
        friendly_name: "Living Room Blinds"
        open:
          service: switch.turn_on
          entity_id: switch.living_room_blinds_up
        close:
          service: switch.turn_on
          entity_id: switch.living_room_blinds_down
```

Configuration variables:

- **covers** array (*Required*): List of your covers.
  - **friendly_name** (*Optional*): Name to use in the Frontend.
  - **open** (*Required*): Defines an [action](/getting-started/automation/) to run to open the cover.
  - **close** (*Required*): Defines an [action](/getting-started/automation/) to run to close the cover.
  - **stop** (*Required*): Defines an [action](/getting-started/automation/) to run to stop the cover.
  - **value_template** (*Optional*): Defines a [template](/topics/templating/) to set the state of the cover. The template should return a number from `0` to `100`. `0` indicates the cover is fully closed, `100` indicates the cover is fully open. Alternatively, the template may return `open` (state will be set to `100`) or `closed` (state will be set to `0`).
  - **entity_id** (*Optional*): Add a list of entity IDs so that the cover only reacts to state changes of these entities. This will reduce the number of times the cover will try to update it's state.
  - **optimistic** (*Optional*): If no template was defined, the cover will have no state (state is `None`). However, if `optimistic` is set to `true`, the cover will assume `open` state immediately after `open` action was executed and `closed` state immediately after `close` action was executed. This option is ignored if `value_template` was defined. Default is `false`.
  - **device_class** (*Optional*): The [type/class](/components/cover/) of the cover to set the icon in the frontend. Default is `window`.


## {% linkable_title Considerations %}

If you are using the state of a platform that takes extra time to load, the template switch may get an `unknown` state during startup. This results in error messages in your log file until that platform has completed loading. If you use `is_state()` function in your template, you can avoid this situation. For example, you would replace `{% raw %}{{ states.switch.source.state }}{% endraw %}` with this equivalent that returns true/false and never gives an unknown result: `{% raw %}{{ is_state('switch.source', 'on') }}{% endraw %}`.

## {% linkable_title Examples %}

In this section you find some real life examples of how to use this cover.

### {% linkable_title Copy switch %}

This example shows a cover that copies another cover.

```yaml
switch:
  - platform: template
    switches:
      copy:
        frinedly_name: "Copy Cover"
        value_template: {% raw %}"{{ states('cover.source') }}"{% endraw %}
        open:
          service: cover.open
          entity_id: cover.source
        close:
          service: cover.close
          entity_id: cover.source
        stop:
          service: cover.close
          entity_id: cover.source
````

### {% linkable_title pilight Cover %}

This example shows a cover that takes its state from a sensor, and opens/closes cover through [pilight](/components/pilight/) platform.

```yaml
switch:
  - platform: template
    switches:
      bedroom:
        friendly_name: 'Bedroom Blinds'
        value_template: {% raw %}"{% if is_state('sensor.bedroom_blinds_proximity', 'on') %}closed{% else %}open{% endif %}"{% endraw %}
        open:
          service: pilight.send
          data:
            protocol: kaku_switch
            id: 10
            unit: 1
            'on': 1
        close:
          service: pilight.send
          data:
            protocol: kaku_switch
            id: 10
            unit: 2
            'on': 1
        stop:
          service: pilight.send
          data:
            protocol: kaku_switch
            id: 10
            'all': 1
            'off': 1
```
