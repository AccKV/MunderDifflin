# Reflection Report

## How the system works

Built 4 agents using smolagents. The main one is the Orchestrator which gets 
the customer request and figures out what to do. Found early on that agents 
kept using wrong item names like "A4 glossy paper" when our db only has 
"Glossy paper" - so added a step that injects the actual inventory list into 
every request. That helped a lot.

Inventory Agent has 4 tools - check_inventory, get_available_items, 
check_item_stock and check_delivery_date. These basically wrap the starter 
helper functions.

Quoting Agent checks past quote history and current cash bal before we commit 
to an order. Also added get_financial_report here which calls 
generate_financial_report from the starter.

Ordering Agent does the actual transaction. Added fuzzy matching to the 
fulfill_order tool so if someone asks for "colored cardstock" it finds 
"Cardstock" in the db. Also auto looks up the price if agent passes 0.

## Eval Results

Ran all 20 requests. Started at $45,059.70 cash bal and ended at $46,492.45 so 
about $1,432.75 in revenue. Inventory dropped from $4,940.30 to $3,507.55 which 
makes sense.

Some examples from test_results.csv:
- Request 1 (office manager, ceremony): ordered glossy paper and cardstock, 
  cash went up showing transaction recorded ok
- Request 2 (hotel manager, parade): balloons and streamers rejected since 
  we dont carry those, correct behaviour
- Request 20 (last request): flyers and tickets rejected, cash stayed same

Requests that worked well were ones asking for cardstock, colored paper, 
glossy paper, banner paper - things we actually stock. Requests for balloons, 
streamers, tickets got rejected which is correct since we dont sell those.

Cash bal changed on quite a few requests which shows ordering is actually 
working and recording transactions.

## What I would improve

1. Quoting agent doesnt always get called before order goes through. 
   Would be better to enforce that as a required step so customers always 
   get a price confirmation first.

2. DB has a min_stock_level column for each item but we never use it. 
   Would be good to add a reorder tool that checks if stock drops below 
   that level and auto triggers a restock order.