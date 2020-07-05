# Custom Links

Custom links allow users to place arbitrary hyperlinks within NetBox views. These are helpful for cross-referencing related records in external systems. For example, you might create a custom link on the device view which links to the current device in a network monitoring system.

Custom links are created under the admin UI. Each link is associated with a particular NetBox object type (site, device, prefix, etc.) and will be displayed on relevant views. Each link is assigned text and a URL, both of which support Jinja2 templating. The text and URL are rendered with the context variable `obj` representing the current object.

For example, you might define a link like this:

* Text: `View NMS`
* URL: `https://nms.example.com/nodes/?name={{ obj.name }}`

When viewing a device named Router4, this link would render as:

```
<a href="https://nms.example.com/nodes/?name=Router4">View NMS</a>
```

Custom links appear as buttons at the top right corner of the page. Numeric weighting can be used to influence the ordering of links.

## Conditional Rendering

Only links which render with non-empty text are included on the page. You can employ conditional Jinja2 logic to control the conditions under which a link gets rendered.

For example, if you only want to display a link for active devices, you could set the link text to

```
{% if obj.status == 1 %}View NMS{% endif %}
```

The link will not appear when viewing a device with any status other than "active."

Another example, if you want to only show an object of a certain manufacturer, you could set the link text to:

```
{% if obj.device_type.manufacturer.name == 'Cisco' %}View NMS {% endif %}
```

The link will only appear when viewing a device with a manufacturer name of "Cisco."

## Link Groups

You can specify a group name to organize links into related sets. Grouped links will render as a dropdown menu beneath a
single button bearing the name of the group.
