# PackSmartAI — Shopify Global Catalog MCP

This integration replaces the single-store product search with Shopify Global Catalog MCP `search_catalog` through `app/api/products/route.ts` and `lib/shopify-global-catalog.ts`. It retains the existing planner, checklist, design, and demo trip bag.

## Configuration

1. Deploy the repository to Vercel or run `pnpm install && pnpm dev` locally.
2. For a quick demo, the server defaults to Shopify's documented **example** UCP agent profile URL. No single-store Shopify Storefront token is needed for this catalog search.
3. For a production agent, host your own UCP agent profile advertising the base catalog search and lookup capabilities and the Shopify global catalog extension. Set `SHOPIFY_UCP_AGENT_PROFILE_URL` in Vercel > Project > Settings > Environment Variables. Refer to https://shopify.dev/docs/agents/catalog/global-catalog and https://shopify.dev/docs/agents/authentication/agent-profiles .
4. Optionally set `SHOPIFY_BUYER_COUNTRY=US` (ISO 3166-1 alpha-2); this is the buyer's shipping country, not their travel destination.
5. Redeploy after changing environment variables.

## How it works

- Packing-list search terms are sent to `/api/products` along with destination, trip type, days, activities and shopping budget.
- The Vercel server calls `https://catalog.shopify.com/api/ucp/mcp` using JSON-RPC `tools/call` and `search_catalog`, with `meta.ucp-agent.profile`.
- Shopify's UCP price amounts are minor units; the server converts them to currency display amounts and filters by the provided per-item price cap.
- Real seller product URLs, images, seller names and variant checkout URLs (when supplied) are shown. The trip bag remains a **simulated** checkout; links open seller sites. No orders or payments are processed in PackSmartAI.
- If Shopify returns an error, the UI shows a retry state rather than fabricated products.

## Important limitations

- Product search and image rendering require a live Shopify service and cannot be verified offline.
- Budget is currently a **per-product maximum**, not a sum-of-cart constraint. The bag's estimated total may exceed it.
- Checkout is not a full UCP checkout integration; multi-merchant carts require separate merchant checkout flows.
- `lib/shopify-storefront.ts` is preserved for reference but is no longer imported by product discovery.
- Never commit private API tokens or `.env.local` files to GitHub.
