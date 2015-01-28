# event-espresso-4-information-on-full-event
As far as I could tell, Event Espresso 4 simply removes the registration form when an event is fully booked, no message is displayed to the end user. I needed that, and got no help in the support forums, so I made my own function doing the job

This is only tested on my limited development setup, so no guarantees that it's working the way you want it to.

## Step 1: Add function to functions.php

First, you need to add this to your themes functions.php:

```
function hent_billettstatus($event_id) {
        global $wpdb;

        $tallene = $wpdb->get_results( $wpdb->prepare (
                                "SELECT DTT_reg_limit, DTT_sold, sum(TKT_qty) TKT_qty, sum(TKT_sold) as TKT_sold from " . $wpdb->prefix . "esp_ticket, " . $wpdb->prefix . "esp_datetime_ticket, " . $wpdb->prefix . "esp_datetime
                                WHERE
                                        " . $wpdb->prefix . "esp_datetime.EVT_ID = %d
                                AND
                                        " . $wpdb->prefix . "esp_datetime.DTT_ID = " . $wpdb->prefix . "esp_datetime_ticket.DTT_ID
                                AND
                                        " . $wpdb->prefix . "esp_datetime_ticket.TKT_ID = " . $wpdb->prefix . "esp_ticket.TKT_ID",
                $event_id
        ));

        $res;
        foreach ($tallene as $tall) {

                $res['DTT_reg_limit'] = $tall->DTT_reg_limit;
                $res['DTT_sold'] = $tall->DTT_sold;
                $res['TKT_qty'] = $tall->TKT_qty;
                $res['TKT_sold'] = $tall->TKT_sold;
                $res['TKT_rem']  = $tall->TKT_qty - $tall->TKT_sold;

                if ($tall->DTT_reg_limit < 0)
                        $res['DTT_reg_limit'] = "Unlimited";

                if ($tall->TKT_qty < 0)
                        $res['TKT_qty'] = "Unlimited";

                if ($res['TKT_rem'] < 0)
                        $res['TKT_rem'] = "Unlimited";

        }

        return $res;

} // hent_billettstatus
```

## Step 2: Edit event template

Then, add the following code to the event template part where you want the message to appear:

```
$billstatus = hent_billettstatus($post->ID);

if ($billstatus['DTT_reg_limit'] <= $billstatus['DTT_sold']) {
        $utsolgt = true;
?>

<div class="tipsboks">
<h2>No available spaces</h2>
<p>Sorry, there are no available spaces for this event.</p>
</div>
```

<?php
}

## (Step 3: Use the function elsewhere)

(The reason the function returns more numbers is that I also use that function to display the number of remaining tickets for events. Feel free to use it as that as well.)
