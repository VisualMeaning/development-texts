## Overview

We improved the accessibility of SMP to help vision impaired people using it with a screen reader. The main thing used is Accessible Rich Internet Applications (ARIA), which  is a set of roles and attributes that define ways to make web content and web applications (especially those developed with JavaScript) more accessible to people with disabilities. It supplements HTML so that interactions and widgets commonly used in applications can be passed to assistive technologies.

Below listed all the specified behaviours when using a screen reader.

## Burger menu
| Object | Behaviour | State | Screen Reader Result |
|-------------|--------|----------|---------|
| Open burger button | Focus  | Not Clicked | Button collapsed not pressed |
| Open burger button | Click  | Clicked | Button expanded pressed |
| Region option | Focus  | Not Clicked | [Region name], button partially pressed |
| Region option | Click  | Clicked | [Region name], button pressed |

## Filters toggles
| Object | Behaviour | State | Screen Reader Result |
|-------------|--------|----------|---------|
| Filter toggle button | Focus  | Not Clicked | Filter toggles list box, not pressed x of y |
| Filter toggle button | Click  | Clicked | Filter toggles list box, pressed x of y |
| Filter toggle options button | Click | Clicked | Options, button pressed |

## Comments panel

| Object | Behaviour | State | Screen Reader Result |
|-------------|--------|----------|---------|
| Open comments panel button | Focus | Not Clicked | Comments panel, button not pressed |
| Open comments panel button | Click | Clicked | Comments panel, button pressed |
| Expand all button | Click | Clicked | Expand all, button pressed |
| Expand all button | Focus | Not Clicked | Expand all, button not pressed |
| Expand arrow button | Click | Clicked | Expand, button pressed |
| Expand arrow button | Focus | Not Clicked | Expand, button not pressed |
| Add comment button | Focus | Not Clicked | Add comment, button not pressed |

## Control panel
| Object | Behaviour | State | Screen Reader Result |
|-------------|--------|----------|---------|
| Search bar | Enter text and return | Search results found | Search results list with x items |
| Search bar | Enter text and return | Search results not found | No results found, earch results list with 0 items |
| Open filters panel button | Focus | Not Clicked | Filters, button collapsed not pressed |
| Open filters panel button | Click | Clicked | Filters, button expanded pressed |
| Open list view panel button | Focus | Not Clicked | List view, button not pressed |
| Open list view panel button | Click | Clicked | List view, button  pressed |
| Expand selected item panel button | Focus | Not Clicked | Expand panel, button collapsed not pressed |
| Expand selected item panel button | Click | Clicked | Expand panel, button expanded pressed |
| Close selected item panel button | Focus | Not Clicked | Close panel, button |

## Intro splash
| Object | Behaviour | State | Screen Reader Result |
|-------------|--------|----------|---------|
| Close button | Focus  | Not Clicked | Button not pressed |

## List view
| Object | Behaviour | State | Screen Reader Result |
|-------------|--------|----------|---------|
| Show full button | Focus  | Not Clicked | Expand, button not pressed |
| Show full button | Click  | Clicked | Expand, button pressed |
