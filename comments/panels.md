# Panels in the SM Platform

Panels in the floating UI are container elements with self-contained content, buttons, links, etc. and do not contain other panels.

Panels in the floating UI float over layers.

Most of the panels that are part of the map or data exploration float over the `.vm-layer.vm-float` layer and should not overlap with each other.

![Floating UI](/assets/panels.png)

Dialog panels float on their own layer over all others with an explicit visual separation (backdrop). They bring a content at the very top and require to be closed to further interact with the rest of the content. There should only be one such panel at a time. Their role is either informative or of a simple form.

![Dialog Panel](/assets/dialog-panel.png)

* Currently overlay panels (Map selection for mobile) live on the same layer as the floating panels and do overlap them, but their role is very similar to that of a dialog panel so it might be worth converting them?

## Component
Each panel or floating element on the layer is denoted with `vm-float-block` with a basic styling (rounded corners, shadow).
``` typescript
export function ExamplePanelComponent(): RNode {
	...
	return <div className="vm-float-block">
		...
	</div>;
}
```
There are some panel components that return content of a panel, in those cases we do not want to return a `<div />` of class `vm-float-block` but the wrapping element should have it.

The panels on the `vm-float` layer exist within `vm-float-containers` sections. Those sections create the layout and should not overlap with each other. 

No panel should live in two sections at a time and any panel flexibility should remain within its containing section ([1](https://github.com/VisualMeaning/sm_platform/pull/746)). Panels within a section should not overlap with each other.

![Panel Containers](/assets/panel_containers.png)

### Dialog Panels
Currently all dialog panels are normal panels absolute/center positioned on their own layer with a div as backdrop.

## Accessibility
For accessibility and screen reading, website content is usually read from top to bottom on page load. Any action of focus or toggling content on and off are also announced to users. In the case of the SM platform where panels are open on demand, it is also important that the newly appearing content is announced. It is highly preferable that all new content is read, but there might be some cases where only the important bits (title/description) is enough.

Another fundamental UX tool is focus management when dealing with an SPA. In the conventional website when you press a link, the page is reloaded and the focus is moved to the beginning, where screen reader's element seeking or tab navigation starts. In an application new content or result of actions appear on the same page. Similar to some HTML components e.g., `<select>`, when you press the element, new elements appear, and you interact with it immediately with keyboard. Similarly, when a panel is opened as a result of button/link click the attention should be moved to it and keyboard navigation starts from its beginning.

To meet the two demands above we have applied a simple solution of moving the focus to a non-focusable panel whenever it is mounted (it appears). The implementation itself may vary to either fit the specificity of the components or some edge cases, but the base would look something like:
``` typescript
export function ExamplePanelComponent(): RNode {
	const ref = useRef<HTMLDivElement>(null);
	...
	useEffect(() => {
		ref.current?.focus()}
	, []);
	...
	return <div ref={ref} className="vm-float-block" tabIndex={-1}>
		...
	</div>;
}
```
Two important bits:
* `tabIndex={-1}` makes the div element programmatically focusable.
* the focus method is called only once after mount. We can simplify this by using a refCallback to have the same effect but useEffect after mount should always work and is more flexible.

There are some more complicated panels such as the SelectedItemPanel (SIP) where interacting with the platform will update the content and will not remount the component. In such case, useEffect is triggered with the dependency list.

### Drawbacks
* what we want here is really to move the [focus positioning](https://github.com/whatwg/html/issues/5326), not change the focused element - document.activeElement should become document.body with the focus point moved to the beginning of the panel.
* focusing non-focusable elements - which is fine after we move away, since they do not appear in the table of focusables.
* with no "single focused container" concept on the platform, we should remove the focused styling of the panel.
