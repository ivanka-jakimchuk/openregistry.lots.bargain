.. _tutorial:

Tutorial
========

Exploring basic rules
---------------------

Let's try exploring the `/lots` endpoint:

.. literalinclude:: tutorial/lot-listing.http
   :language: javascript

Just invoking it reveals empty set.

Now let's attempt creating a lot:

.. literalinclude:: tutorial/lot-post-attempt.http
   :language: javascript

Error states that the only accepted Content-Type is `application/json`.

Let's satisfy the Content-type requirement:

.. literalinclude:: tutorial/lot-post-attempt-json.http
   :language: javascript

Error states that no `data` has been found in JSON body.


.. index:: Lot

Creating lot
------------
You can create lot with relatedProcesses at once.
It`s called a batch mode.

.. literalinclude:: tutorial/lot-post-with-rPs.http
   :language: javascript


Let's create lot with the minimal data set:

.. literalinclude:: tutorial/lot-post-2pc.http
   :language: javascript


The object you're trying to add initially receives `draft` status. You should manually switch this object from `draft` to `composing` (2 Phase Commit mechanism) so that to add the auction conditions (value.amount, minimalStep.amount, etc.):

.. literalinclude:: tutorial/lot-to-composing.http
   :language: javascript

You see that `lot.auctions` structure has been added with the set of auto-generated data. 

Now let's add extra auction conditions. Note that the information is being added to each of three auctions one by one:

.. literalinclude:: tutorial/compose_lot_patch_1.http
   :language: javascript

.. literalinclude:: tutorial/compose_lot_patch_2.http
   :language: javascript

Now let's add relatedProcesses:

.. literalinclude:: tutorial/add_related_process_1.http
   :language: javascript

To enable further manipulations with the lot, its status should be manually switched to `verification`.

.. literalinclude:: tutorial/lot-to-varification.http
   :language: javascript

After concierge switch lot status to `pending`.
Owner has an opportunity to switch lot status to `active.salable`.
The procedure will be formed automatically after this action.:

.. literalinclude:: tutorial/owner-patch-lot-to-active.salable.http
   :language: javascript

Success! Now we can see that new object was created. Response code is `201`
and `Location` response header reports the location of the created object.  The
body of response reveals the information about the created asset: its internal
`id` (that matches the `Location` segment), its official `assetID` and
`dateModified` datestamp stating the moment when asset was last
modified. Note that lot is created with `pending` status.

Let's access the URL of the created object (the `Location` header of the response):

.. literalinclude::tutorial/blank-lot-view.http
   :language: javascript

.. XXX body is empty for some reason (printf fails)

We can see the same response we got after creating lot.

.. literalinclude:: tutorial/lot-post-2pc.http
   :language: javascript

Let's see what listing of lots reveals us:

.. literalinclude:: tutorial/initial-lot-listing.http
   :language: javascript

We do see the internal `id` of the lot (that can be used to construct full URL http://lb.api-sandbox.registry.ea2.openprocurement.net/api/2.4/lots/8286cc7863814271afe23cb2646237ed`) and its `dateModified` date stamp.

Let's try creating another lot:

.. literalinclude:: tutorial/create-second-lot.http
   :language: javascript

.. XXX patching lot to pending.dissolution

And again we have `201 Created` response code, `Location` header and body with extra `id`, `lotID`, and `dateModified` properties.

Switch second lot to 'composing' status':

.. literalinclude:: tutorial/second-lot-to-composing.http
   :language: javascript

Let's check what lot registry contains:

.. literalinclude:: tutorial/listing-with-some-lots.http
   :language: javascript

And indeed we have 2 lots now.


Modifying Lot
-------------

Let's update lot description:

.. literalinclude:: tutorial/patch-lot.http
   :language: javascript

.. XXX body is empty for some reason (printf fails)

We see the added properties have merged with existing lot data. Additionally, the `dateModified` property was updated to reflect the last modification date stamp.

Checking the listing again reflects the new modification date:

.. literalinclude:: tutorial/lot-listing-after-patch.http
   :language: javascript

Deleting Lot
------------

Let's delete lot:

A document with the `documentType: canellationDetails` has to be added first:

.. literalinclude:: tutorial/add_cancellation_docs.hhtp
   :language: javascript

So now lot can be easily deleted:

.. literalinclude:: tutorial/lot-delete-2pc.http
   :language: javascript

Integration with assets
-----------------------


Concierge operations
--------------------

For lot to be formed, you need to specify id of the asset which is to be included 
in that lot. If this assets is available, it will be attached to lot 
and status of a lot itself will be changed to `pending`. The given lot becomes:

.. literalinclude:: tutorial/lot-after-concierge-patch-pending-2.http
   :language: javascript

In case of this assets is unavailable, status of the current lot will turn to `invalid`
The given lot becomes:

.. literalinclude:: tutorial/lot-after-concierge-switch-to-invalid.http
   :language: javascript

When bot finds that status of lot is `pending.deleted`, it
turns status of the asset being attached to that lot to `pending`. Status of the lot itself will become `deleted`.
The given lot becomes:

.. literalinclude:: tutorial/lot-delete-3pc.http
   :language: javascript

When bot finds that status of lot is `pending.dissolution`, it
turns status of the asset being attached to that lot to `pending`. Status of the lot itself will become `dissolved`.
The given lot becomes:
   
.. literalinclude:: tutorial/lot-after-concierge-patch-lot-dissolved.http
   :language: javascript

When bot finds that status of lot is `pending.sold`, it
turns status of the asset being attached to that lot to `complete`. Status of the lot itself
turns to `sold`. The given lot becomes:

.. literalinclude:: tutorial/lot-after-concierge-patch-lot-sold.http
   :language: javascript
   
Convoy operations
-----------------

When the procedure is successfully created, lot status changes to `active.auction`: 

.. literalinclude:: tutorial/switch-lot-active.auction.http
   :language: javascript


In case of that lot has not been sold (either `contract` has become `unsuccessful` or a procedure has received `cancelled` status or `unsuccessful`) , its status becomes `pending.dissolution`. The given lot becomes:

.. literalinclude:: tutorial/lot-after-convoy-patch-auction-cancelled.http
   :language: javascript


When contract has been successfully created within the Module of Contracting, lot's status turns to `active.contracting`, after what lot becomes `complete`:

.. literalinclude:: tutorial/lot-after-convoy-patch-auction-complete.http
   :language: javascript

When contract reaches `terminated` status, lot automatically becomes `pending.sold`, after what lot becomes `complete`:

.. literalinclude:: tutorial/lot-after-caravan-patch-contract-complete.http
   :language: javascript
