# Mobiliaire
A lightweight property inventory helper

## This repository has moved to GitLab. Please visit [Cordelya's GitLab](https://gitlab.com/cordelya) and you'll find the active project there.

> mobiliaire (n.): Middle French form of "mobiliary"; pertaining to furniture or movable property

*built for/on Django 3.1*

[Changelog](https://github.com/Cordelya/mobiliaire#Changelog)

This self-hosted app is for assisting SCA branches in managing both annual inventories and pack-in/pack-out at events. It is designed to be installed on a portable machine - to be taken to events and serve up web pages with inventory information without needing access to the Internet. A [$35 Raspberry Pi](https://www.raspberrypi.org/products/) fits the bill nicely here, but take my advice and spend the extra $10 for a [case](https://www.raspberrypi.org/products/raspberry-pi-4-case/) if you go that route. A tablet with Termux or a similar application may work as well (tested on a Samsung Galaxy Note 8).

There is now [a fork of this repository just for Raspberry Pi](https://github.com/Cordelya/mobiliaire-raspi) that includes support for submitting new photos via [picamera](https://github.com/waveform80/picamera).

See it in action at https://cordelya.pythonanywhere.com (note: this will be whatever's in the main branch - changes in dev branches won't be available)

Want to try it out? 

````
$ git clone https://github.com/Cordelya/mobiliaire.git
````
[See a more detailed procedure.](https://github.com/Cordelya/mobiliaire/wiki/getStarted)

Inventory Items are grouped into Boxes (which can be literal or virtual boxes), which are then further grouped into Warehouses. In this context, a warehouse is a geographic location where a box or boxes are stored. A branch cargo trailer full of items is a Warehouse. Examples of virtual boxes include the "loose in the trailer" box, the "driver-side cargo rack" box, or the "gold-key-closet" box. 

Key: (N) = not yet implemented

### Put Data in, Get Data Out ###
* Tracks the replacement value of each item. Front-end displays total aggregate value on index, and sub-values on Warehouse and Box detail pages. Why? Insurance.
    * Value data broken down by consumable/not consumable and other breakpoints is planned for a future release.
* PDF/Excel/CSV/Print export of any data on reports pages. Each table has its own set of export buttons, allowing you to build a custom report. You can filter tables before exporting and only export the filtered rows. You can sort columns before exporting and the sort order will be reflected in the result.
   * Page /reports/full/ has all data presented, requiring the report-generator to click only four buttons (warehouse list, boxes by ID, boxes by name, and items). 
   * At this time, print or pdf reports with each box separated can be done via the reports/wh/ page but it requires clicking each box's export button. One-click button triggering is deferred until after version 1.0 is released.
* CSV import is implemented but it is up to the site admin to set up the import config for each csv file to be imported. This means you can, in theory, config the importer to import your entire inventory spreadsheet, no matter what condition it is in. For the rest of us, start with staff, warehouses, and keywords, then do boxes, then items and attach your items to the appropriate box and keywords.
* Export database as database-friendly backup file (excluding the auth.permission and contenttypes tables will reduce errors on db load later. If you exclude the auth.permission table you will need to create a new superuser when you load this exported data into a fresh install. If you're loading the data into a read-only instance, you don't need to create a superuser - nobody will be able to log in to the admin side.)
~~~~bash
$ ./manage.py dumpdata --exclude auth.permission --exclude contenttypes > db.json
~~~~
* A dashboard showing various information about the state of your inventory is planned for a future release.

### Faster Pack-In/Out and Inventories ###
* Set keywords for each item to make search results filter-able. Add as many keywords as applicable. Suggested keywords include "Category: <category>" for categories like "kitchen" "archery" "heraldic" etc; "Color: <color>" to classify items by their main color(s); "Material: <material>" to classify by material (glass, metal, plastic, etc)
    * Keywords show on item detail page
    * Keyword detail pages list all items associated with a particular keyword
* Pack-in/out friendly search: when an item is needed or arrives for packing, begin search with general item description (ie "ladle"), filter items by keyword (Category: Kitchen, Material: Metal, etc.) until matching item is identified (verify visually via inline photo) and place item in indicated box. Can go very quickly if a person familiar with the system is staffing the search terminal. For best results, limit search query to a single word (complex search planned for after version 1.0 is released)
* Items report table (/reports/items/) is also fully searchable, but does not display pictures and does not include keywords. Would need to click through to each search result to verify. Displaying item image files via Bootstrap modal is planned for after 1.0 is released.
* Search bar allows simple searching (limit your query to a single word) across Warehouses, Boxes, Items, and Keywords, showing results from each.
* Did you rearrange? Move items from one box to another by updating the current box's end date and creating a new items_in_boxes record for the new box, leaving the end date blank. Item's box history will be preserved. Front end shows only current location. 
    * When someone says, "Doesn't this item belong in {former box}?" admin can check the item's box history and see that it used to be in box A but now is assigned to box B.
    * If you know when an item was acquired, you can back-date the item's oldest box history record (the date_from field) to capture that fact. Get with your financial record-keepers. Display of such metadata is planned for a future release.
 
### Consumables Differentiation ###
* Tracks whether items are consumable or not
    * (N) List consumable items, per box, for printing to allow event staff or annual inventory staff to verify quantities remaining and make notes.
        * A stopgap is available on the box report page - type "True" in the search box on each table and click each table's "PDF" button and you have a list. 
        * Or, visit the "Consumables" page, sort the far-right column to display items with "buy soon" or "buy *n* items" rows at the top, then click "print" and limit your print job to the pages showing less than 100% remaining. 

### Image Support ###
* Warehouse detail page shows photo associated with specified warehouse, if set. This could be an image of an individual's or officer's armory, or a photo of the physical warehouse.
    * This defaults to "warehouse.png" to facilitate using a placeholder file.
* Box listing on warehouse detail, box list, and box detail includes photo of specified box, if set. 
    * Defaults to "box.png" to facilitate using a placeholder file.
* Item listing on box detail, item list, and item detail shows item photo, if set. Set photo by saving image file to /static/inv/img and recording the image's filename in the item's record. Recommended image file naming convention: by item ID, item name, or combo. Image name field and image name (including extension, but excluding path) must match exactly. Run python manage.py collectstatic after adding any files to the static/ folder tree. 
    * If image is not set, helper text asks viewer to snap a photo of the item and email it to the inventory management contact with the item's id number in the subject line (item id number is displayed). Adding photos via webcam control is planned for a release after 1.0.
        * There's no placeholder for contact information for the inventory manager, but the global footer would be a good place to put that information. The global footer's markup is in templates/inv/base.html, in between `<footer ...>` and `</footer>`
    * New Item records do not set a photo filename by default, but this could be accomplished by editing the Items model in inv/models.py, where 'item.png' is the name of your default item image file.
 ~~~~python
 img/models.py
 ...
 class Items(models.Model):
 ...
 item_img = models.CharField(max_length=50, blank=True, default='item.png')
 ~~~~
 
This app is built on the well-established and well-documented Django framework. If you're not sure how to do something, try checking the Django documentation pages first.

# Changelog #
## Version 1.0.1 released October 15, 2020
* Unification of template styling so that warehouse/box and warehouses/boxes/items appear the same. 
* Keyword view styling updated. 
* Items.html keyword filtering styling changed. 
* "Missing photo" placeholder styling changed in several locations.
* index.html total value calculation in views.py corrected
* index.js updated, splitting DataTables initializer into two views, t1 and t2. Currently, t1 is applied to all reports tables, and t-2 is applied to the table on the consumables page - to facilitate showing the dt-SearchBuilder extension for smarter filtering.
* index.js updated to reflect element changes in items.html (from `<li>` to `<button>`)

## Version 1.0 released October 5, 2020
* Initial Stable Release incorporating all work since project started.
