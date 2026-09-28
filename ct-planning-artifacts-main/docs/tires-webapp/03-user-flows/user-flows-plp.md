# Product Listing Page
On this page, users are presented with a selection of tires that match their search criteria. The page provides key product information and features that allow users to browse, compare, filter, sort, and add tires to their cart.

## Product Tiles
Clicking a product tile redirects the user to the Product Details Page for more information.

Each product tile includes:
- Product image, name, brand logo, item number, feature badge, price, tire size, season, load index, speed rating, warranty mileage, run-flat status, and promotions.
- Quantity selector with increment/decrement buttons.
- Add to Cart and Compare options (up to 4 products can be selected for comparison).
- A variant for displaying staggered fitment details.

## Compare Functionality
- Products added for comparison are saved in Redux state and persisted in localStorage for session continuity.
- Selected products appear in a collapsible compare tray at the bottom of the page.

## Added to Cart Drawer
- Displays products added to the cart.
- Opens automatically when a user adds a product to the cart.
- Cannot be manually opened by the user.

## Sort By (Dropdown)
Users can sort the product list by:
- Top Selling
- Price (Low → High / High → Low)
- UTQG (High → Low)
- Promotion
- On mobile, sort functionality is available inside the filter drawer; the dropdown on the page (outside filters) is hidden.

## Filtering
- Filters are available for Wheel Type (For staggered fitment), Price range, Brand, Season, Compatibility, Run Flat, Load Index, Speed Rating.
- Filters are collapsible on mobile (drawer) for better UX.
- On mobile:
  - Sort By functionality is inside the filter drawer.
  - Season quick-filters are toggle buttons, horizontally scrollable.
  - The filters button shows the number of filters applied in a small red circle.
- Filter state is persisted in the URL for shareable links and session continuity.
- A Clear All Filters link is provided.

#### Scenario 1: User applies a price filter on desktop
- Checks the price range filter "$0 - $49".
- Filter is applied.
- Clear All Filters link becomes active; clicking it removes all filters.

#### Scenario 2: User applies a price filter on mobile
- User clicks the filters button.
- Filter drawer opens, and price filter "$0 - $49" is selected.
- Filters button updates to show number of filters applied (1).
- Clear All Filters link becomes active; clicking it removes all filters.
- Filter is applied, drawer closes.

#### Scenario 3: User clicks compare button in the compare tray with only 1 product selected
- Error message displayed: "Please select at least 2 tires to compare."

#### Scenario 4: User adds products to compare
- Products appear in the compare tray.
- Up to 4 products can be selected.
- Clicking Compare redirects to Compare Tires Page.

#### Scenario 5: User tries to add more than 4 products to compare
- Compare tray displays error: "You can only compare up to 4 products. Remove one or more products before adding another to compare."

#### Scenario 6: User tries to reduce quantity below 1
- Error message displayed: "Must select at least one tire."

#### Scenario 7: User adds a product to cart
- User selects a quantity and clicks Add to Cart button
- Selected items are added to the cart.
- Added to Cart drawer opens automatically, showing the items added to cart.
- User can continue browsing or navigate to the cart to review selections.

#### Scenario 8: User interacts with an out-of-stock product
- Quantity selector is removed.
- Add to Cart button replaced with View Details.
- Label Temporarily Out of Stock is displayed in red.