# inventory
A lightweight property inventory helper

## Still in development (v0.\*). Main branch is generally stable. 1.0 coming soon to a "stable" branch near you.

*built for/on Django 3.1*

This app is for assisting SCA branches in managing both annual inventories and pack-in/pack-out at events. It is designed to be installed on a portable machine - to be taken to events and serve up web pages with inventory information without needing access to the Internet. A [$35 Raspberry Pi](https://www.raspberrypi.org/products/) fits the bill nicely here, but take my advice and spend the extra $10 for a [case](https://www.raspberrypi.org/products/raspberry-pi-4-case/) if you go that route. A tablet with Termux or a similar application may work as well.

See it in action at https://cordelya.pythonanywhere.com (note: whatever's in the main branch - changes in dev branches won't be available)

Want to try it out? 

````
$ git clone https://github.com/Cordelya/inventory.git
````
(Scroll to the bottom for a more detailed guide if you're not familiar with the Django framework.)

Inventory Items are grouped into Boxes (which can be literal or virtual boxes), which are then further grouped into Warehouses. In this context, a warehouse is a geographic location where a box or boxes are stored. A branch cargo trailer full of items is a Warehouse. Examples of virtual boxes include the "loose in the trailer" box, the "driver-side cargo rack" box, or the "gold-key-closet" box. 

Key: (I) = implemented
     (N) = not yet implemented

(I/N) notations updated 9/28/2020.

* (I) Tracks whether items are consumable or not
* (I) Tracks the replacement value of each item. Front-end displays total aggregate value on index, and sub-values on Warehouse and Box detail pages.
* (I) Set keywords to make search results filter-able. Add as many keywords as applicable. Suggested keywords include "Category: <category>" for categories like "kitchen" "archery" "heraldic" etc; "Color: <color>" to classify items by their main color(s); "Material: <material>" to classify by material (glass, metal, plastic, etc)
  * (I) Keywords show on item detail page (implemented 9/20/2020)
  * (I) ~~Keyword detail pages list all items~~ Keyword filtering available on /items. Page that focuses on keywords deferred to after 1.0.
* (I) Move items from one box to another by updating the current box's end date and creating a new items_in_boxes record for the new box, leaving the end date blank. Item's box history will be preserved. Front end shows only current location.
* Reports: 
  * (I) PDF export with entire inventory arranged by warehouse and box with no orphaned tables or optionally starting a new page every box.
     * All tables needed for full report are displayed on /reports/full/ requiring the report-generator to click four buttons (warehouse list, boxes by ID, boxes by name, and items). 
     * At this time, print or pdf reports with each box separated can be done via the reports/wh/ page but it requires clicking each box's export button. One-click button triggering is deferred until after 1.0.
  * (I) PDF export per table on all 3 current /reports/ directory views, via dataTables. Has dependencies not included/tracked in this repo
  * (I) export entire inventory to .csv for spreadsheet-friendly backup. Filter by keywords, Warehouses, boxes. Group by Warehouse or Box (NOTE: .csv does not support multiple sheets per file, so if you request grouping by Warehouse or by Box, you will receive multiple .csv files - one for each group object.)
     * You can grab individual boxes, a master list of items, a list of boxes, and a list of warehouses.
  * (I) export database as database-friendly backup file (available on command-line)
  * (N) consumable items, per box, for printing to allow event staff or annual inventory staff to verify quantities remaining and make notes.
     * A stopgap is available on the box report page - type "True" in the search box on each table and click "PDF" and you have a list. 
  * (I) consumable items, filtered to show items which are running low, to use as a shopping list for replenishment
* (I) Pack-out friendly search: when item arrives for packing, ~~begin search with general item description (ie "ladle"),~~ filter items by keyword (Category: Kitchen, Material: Metal, etc.) until matching item is identified (verify visually via inline photo) and place item in indicated box. Can go very quickly if a person familiar with the system is staffing the search terminal.
     * Items report table (/reports/items/) is fully searchable, but does not display pictures and does not include keywords. Would need to click through to each search result to verify. Displaying item image files via Bootstrap modal is planned for after 1.0 is released.
     * Searching by item name not yet implemented. That search will use same template as item gallery and include the same "onClick" javascript keyword filtering.
 * (I) Warehouse detail page shows photo associated with warehouse, if set. This could be an image of an individual's or officer's armory, or a photo of the physical warehouse.
     * Defaults to "warehouse.png" to facilitate using a placeholder file.
 * (I) Box listing on warehouse detail, box list, and box detail includes photo of box, if set. 
     * Defaults to "box.png" to facilitate using a placeholder file.
 * (I) Item listing on box detail, item list, and item detail shows photo, if set. Set photo by saving image file to /static/inv/img and recording the image's filename in the item's record. Recommended image file naming convention: by item ID, item name, or combo. Image name field and image name (including extension, but excluding path) must match exactly. Run python manage.py collectstatic after adding any files to the static/ folder tree. 
     * If image is not set, helper text asks viewer to snap a photo of the item and email it to the inventory management contact with the item's id number in the subject line (item id number is displayed). Adding photos via webcam control is planned for a release after 1.0.
 
 ## How to get it: 
 (note: I haven't run through this procedure from start to finish yet - let me know if I've missed a step somewhere)
 Dependencies: 
 * Python3[.6|.7|.8] (get the latest you can, but 3.6 is still supported if that's what you've got.)
 * git - installable via your package manager
 * Pip (comes with Python. You may need to use "pip3" to invoke)
 * Django - installable via pip 
 * DataTables and Bootstrap 4. | templates/inv/base.html is set up to load local js/css assets for offline operation by default and by design. You can either grab the compiled assets from their respective websites or change the refs to CDN refs, both in the CSS refs in the header and the JS refs near the bottom of templates/inv/base.html.
     * DataTables has a custom asset builder at https://datatables.net/download/. At minimum, you need:
          * Bootstrap 4 in the styling category
          * jQuery and DataTables in the packages category
          * Buttons (and sub extensions per your button needs)
          * Any other dt extensions you want access to
     * Both Bootstrap 4 and the keyword filtering function also use jQuery.
     
 in the folder where you want to install the app (it comes with a top-level folder called "inventory"), do:
 
 ````
 $ git clone https://github.com/Cordelya/inventory.git
 ````
 
Copy credentials-example.py to credentials.py and enter a generated secret key in the file, inside the single quotes, overwriting "paste your secret key here". Search "django secret key generator" if you need help with that.

````
$ python manage.py migrate
$ python manage.py createsuperuser # create a user account so you can add items to the database using the admin web interface. Don't forget those login details!
$ python manage.py runserver
````

Head to 127.0.0.1:8000/admin and log in

Begin by adding at least one warehouse. Add at least one box to one warehouse. Add some keywords and at least one staffmember. Add at least one item (and associate it with a box and keywords in the same form).

Suggested keywords include:
* categories structured as Category: Category, so kitchen items might use a category keyword of Category: Kitchen
* colors structured as Color: Color, so an item that is blue and red might use Color: Blue and Color: Red
* materials structured as Material: Material, so an item that is glass might use Material: Glass

You can add as many keywords as you want to each item. If you want to mark some items that need attention, you could assign them a "TODO" keyword and then filter on the "TODO" keyword later on.

A note about items and quantities:

If you have 15 non-disposable water pitchers that are all the same, you can put them in one item record. Set the quantity to 15 and leave the "percent remaining" field at 100. 
If you have 200 disposable plastic cups, but a "full" supply of cups is 500, you can enter 500 as the quantity, check the "consumable" box, and then enter 40 in the "percent remaining" field. 

If you have a roll of plastic cling wrap, enter 1 as the quantity, and estimate how much is left on the roll for the "percent remaining" field. Check the "consumable" box.
When you view the consumables shopping list, you will see all of the consumable items, their "full" quantity, and their percent remaining. If the item count is greater than 1, the quantity needed will be calculated based on the "full" quantity and percentage remaining so your restocking shopper will know how many of each item they need to buy.

This app is built on a well-established and well-documented framework. If you're not sure how to do something, try checking the Django documentation pages first.
