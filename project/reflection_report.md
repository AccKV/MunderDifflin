# Reflection Report

## How the system works and why this architecture

Built 4 agents using smolagents. Chose 3 worker agents because the tasks split 
naturally into 3 separate concerns - checking stock, pricing, and placing orders. 
Combining them into fewer agents would make each agent responsible for too many 
things and harder to debug. Adding more agents wouldnt make sense since there 
are only 3 distinct job types.

The Orchestrator is a CodeAgent rather than a ToolCallingAgent because it needs 
to write and run Python code to coordinate the worker agents. It decides which 
agents to call based on what the request needs - for most requests it calls 
inventory first to check stock, then quoting for pricing context, then ordering 
to finalize. For requests where items arent in stock it stops after inventory.

Found early on that agents kept using wrong item names like "A4 glossy paper" 
when our db only has "Glossy paper" - so added a step that injects the actual 
inventory list into every request. That fixed most of the name matching issues.

Inventory Agent has 4 tools - check_inventory, get_available_items, 
check_item_stock and check_delivery_date wrapping get_all_inventory, 
get_stock_level and get_supplier_delivery_date from the starter.

Quoting Agent uses get_quote_history, get_current_cash and get_financial_report 
wrapping search_quote_history, get_cash_balance and generate_financial_report.

Ordering Agent uses fulfill_order which wraps create_transaction and get_stock_level.
Added fuzzy name matching so "colored cardstock" finds "Cardstock" in the db.
Also auto looks up the unit price if the agent passes 0.

## Eval Results

Ran all 20 requests from quote_requests_sample.csv. Started at $45,059.70 cash 
bal and ended at $46,492.45 so about $1,432.75 in revenue. Inventory dropped 
from $4,940.30 to $3,507.55.

A key strength was the fuzzy item name matching in fulfill_order. Without it 
almost every order failed because customers say "glossy A4 paper" but our db 
has "Glossy paper". Requests 1, 3, 6, 9 all succeeded because of this matching.

Another strength was correctly rejecting items we dont carry. Requests for 
balloons, streamers, tickets and flyers all got declined with a proper message 
since those arent in our catalog. This is correct behaviour and the system 
handled it consistently across all 20 requests.

The cash bal changed across multiple requests showing transactions are actually 
being recorded in the db. Requests 1, 3 and 9 all show different cash levels 
in test_results.csv confirming this works.

## What I would improve

1. The quoting agent doesnt always get called before the order goes through. 
   A proper fix would be to add a mandatory quoting step in the Orchestrator 
   before any fulfill_order call. This would mean every customer gets a price 
   confirmation before the order is committed, which is better practice and 
   reduces the chance of wrong pricing going through.

2. The db has a min_stock_level column for each item but we never use it. 
   Would add a check_reorder tool to the inventory agent that compares current 
   stock against min_stock_level after each sale. If stock drops below the 
   threshold it would automatically trigger a stock_orders transaction to reorder 
   from the supplier. This would prevent stockouts and make the system more 
   autonomous.