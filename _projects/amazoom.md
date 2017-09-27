---
layout: project
title:  Amazoom Automated Warehouse
date:   2017-07-12 17:50:00
authors: [C. Antonio Sánchez]
categories: [project, automated warehouse]

---

# Amazoom: Automated Warehouse Proposal
{:.no_toc}

For this course project, you will *design* and create a *real-time simulation* of the system software that runs an automated warehouse.  You are responsible for producing a [design document](https://en.wikipedia.org/wiki/Software_design_description) that describes the system architecture and information flow for your automated warehouse design, as well as a multi-process, multi-threaded simulation of the software to prove to Amazoom that your design is safe, efficient, and will satisfy all their needs.

The project can be done individually, in pairs, or in groups of three.  The more members on your team, the more is expected of you.

{% include toc.html %}

## Introduction

Amazoom is the largest internet-based retailer in the world, selling and shipping everything from bananas to laptops to engagement rings.  They're currently looking to cut costs by automating all of their warehouses, but they do not have the in-house expertise to design such a thing.  They *just* put out an open call for proposals.

Your engineering firm has never tackled such a project before, but you're confident that with your new skills in *System Software Engineering*, you have a shot and winning the multi-billion dollar contract.  Not only do you plan to submit a configurable and scalable design, but also to wow them with a simulation of it up and running.  This will help prove that you are the right team for the job.

## Warehouse Function

### Layout

Amazoom's warehouses are laid out on a grid, with long aisles of shelving.  Their products are [randomly distributed](https://www.youtube.com/watch?v=5TL80_8ACPc) on the shelves (e.g. pickles can be next to perfume) to allow for faster collection of items when putting together an order.  The warehouse grid and shelves are labelled so that products can be easily located.  For example, a box of Kleenex might be stored at (A, 3, right, 6), which corresponds to column A, row 3, on the right-hand side, and the 6th shelf from the bottom. Each shelving space  -- corresponding to a single coordinate -- has a limited weight capacity for storing items.

![sample warehouse layout]({{site.url}}/assets/projects/warehouse_layout.png)

The warehouse has a loading bay with a fixed number of docks.  Any dock can be used for either incoming inventory (to restock), or for outgoing deliveries.

### Central Computer

The warehouse has a central computer system that is used to keep track of:
- a database of products and inventory
- the locations of inventory within the warehouse
- list of orders received, orders ready for delivery, and orders out for delivery
- the arrival/departure of delivery and inventory trucks

When orders are placed, the computer plans the routes to take to collect items.  When restocking, the computer plans the route and locations for new items to be placed.

Orders come in from a single *remote webserver*.  If the warehouse has all the items requested, it fulfils the order by:
- updating the inventory list to reflect the sale
- inserting the order into a queue to be collected for delivery

All items in the order are collected and brought to the next waiting delivery truck.  Once on the truck, the central computer is notified that the order is ready for delivery.

The computer also provides a user-interface where the warehouse manager can
- query the status of an order
- check the number in stock of an item
- get alerts about low-stock items
- order more of an item that is low-in-stock from the supplier

### Delivery and Restocking

A delivery or restocking truck can dock at any available slot in the loading bay.  If no slot is available, the truck waits until one becomes free.

Once docked, a delivery truck notifies the central computer of its arrival, and waits at the dock until it is "full enough" to warrant a delivery.  The truck has a limited cargo weight capacity for holding items.  An order should not be put onto the truck unless the entire order can fit.  Once the truck leaves, it notifies the central computer and the dock becomes available for any waiting vehicle.

When a truck bringing in new stock docks in the loading bay, it notifies the central computer of its arrival.  It then remains docked until all items are removed.  Once empty, it notifies the central computer and leaves.

### Automation

Amazoom wants to automate the collecting and loading of orders for delivery, and the restocking of items on the warehouse shelves.

Your engineering firm has already designed the perfect robots for the job.  They can be wirelessly told where to go, have advanced object recognition and collision avoidance capabilities, and are both extremely strong and dexterous, capable of picking up any item that Amazoom sells.  They do have a certain carrying capacity by weight, however, limiting the number of items they can transport at a time.

Your job, as the new system software engineer, is to design and implement the software that runs the entire warehouse operation.  This includes software processes for:
- The central warehouse computer
- The robots
- The remote webserver that places orders

## Getting Started

Rather than jumping straight into the design of a full complex system, start with a simpler system and build up as you get things working.  For example, try beginning with only a fixed warehouse layout and a single robot and program your robot process to navigate around the warehouse without crossing through shelving units.  Then slowly add the central computer, product database with one or two items, interactions with deliveries, user-interface, remote machine for orders, and so on.

## Requirements

### Individual Submission

If submitting the project as an individual, you are required to design and implement the system as described above.  For full marks, your warehouse automation must support

- a fixed warehouse layout
- a fixed number of warehouse robots, at least 4
- a fixed number of products, at least 5


### Pair Submission

For paired submissions, we extend the warehouse design to include flexible layouts, on-the-fly robot acquisitions, an updateable product database, and multiple incoming order connections.

### Team of Three Submission

For teams of three, we extend the paired submission requirements to include a network of warehouses.



## Grading
