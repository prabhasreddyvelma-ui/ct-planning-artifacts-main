# Components Behavior

## Dropdowns
All dropdowns in the application, except the **Sort By** dropdown on the Products Listing Page, should function as comboboxes:

- Users can type to search/filter options.
- Users can select an option from the list.
- The component supports keyboard navigation (up/down arrows, Enter to select).
- The dropdown should be accessible (ARIA attributes applied).

**Note**: This applies to all dropdowns throughout the application.

# Standalone Pages

| Page Name | Route | Additional Notes |
|-----------|-------|------------------|
| Tires Home | / |
| Size Selection Page | /sizeSelection | This page cannot be accessed directly without Year, Make, Model details in the URL. Users will be redirected to the Tires Home Page if these details are missing. |
| Products Listing Page | /SearchResultsByVehicle OR /SearchResultsBySize | This page can be accessed through a Vehicle Search or Tire Size Search |
| Product Details Page | /Product | This page cannot be accessed directly without a product context (Item number) in the URL. Users will be redirected to the Tires Home Page if no product context is provided. |
| Compare Tires | /Tirecomparison |
| Tire Center Locator | /TireCenterLocator | This feature can also be accessed via a modal when the user is changing the Tire Center or navigating the Schedule an Appointment flow. |

# Common Functionality

## Header
A standard responsive header with the following links:

- **Sign In/Register** (visible to anonymous users): Clicking on this link will redirect the user to external Sign In/Register Page.
- **My Account** (visible to signed-in users): Clicking on this link will redirect the user to external My Account/Profile Page.
- **Tire Center / Selected Tire Center Name**: If no Tire Center is selected, clicking on this link will redirect the user to the **Tire Center Locator** standalone page. If a tire center is already selected, it will instead open the **Tire Center Locator** in a modal.
- **Cart** - Clicking on this link will redirect the user to external cart page.

| Resolution | Design Reference | Scenario |
|------------|------------------|----------|
| Mobile | "docs/tires-webapp/02-design/mobile/images/Common Functionality/Header/Header - Signed Out.jpg" | Signed Out |
| Mobile | "docs/tires-webapp/02-design/mobile/images/Common Functionality/Header/Header - Signed In.jpg" | Signed In |
| Desktop | "docs/tires-webapp/02-design/desktop/images/Common Functionality/Header/Header - Signed Out.png" | Signed Out |
| Desktop | "docs/tires-webapp/02-design/desktop/images/Common Functionality/Header/Header - Signed In.png" | Signed In |

## Main Site Navigation
Responsive menu for site-wide navigation with the following links:

- **Tires Home**: Clicking on this link will redirect the user to Tires Home Page
- **Batteries**: External link
- **Care & Maintenance**: External link
- **Warranty**: External link
- **Costco.com**: External link
- **Schedule an Appointment**: Clicking on this link will open the **Schedule an Appointment** Modal
- **Locations**: External link

**Note**: On mobile, the main navigation will be collapsed into a **'Shop'** hamburger menu, with the **Schedule an Appointment** link remaining visible outside the menu, as shown in the mobile layout.

| Resolution | Design Reference | Scenario |
|------------|------------------|----------|
| Mobile | "docs/tires-webapp/02-design/mobile/images/Common Functionality/Main Site Navigation/Main Site Navigation.png" | Shop Menu - Collapsed
| Mobile | "docs/tires-webapp/02-design/mobile/images/Common Functionality/Main Site Navigation/Shop Offcanvas Menu - Slide In from the left.png" | Shop Menu - Expanded |
| Desktop | "docs/tires-webapp/02-design/desktop/images/Common Functionality/Main Site Navigation/Main Site Navigation.png" |

## Footer
Standard site footer with links and branding

| Resolution | Design Reference |
|------------|------------------|
| Mobile | "docs/tires-webapp/02-design/mobile/images/Common Functionality/Footer/Footer.png" |
| Desktop | "docs/tires-webapp/02-design/desktop/images/Common Functionality/Footer/Footer.png" |

## Tire Center Locator Modal
The functionality will mirror the **Tire Center Locator** standalone page, but when the user changes the Tire Center or accesses it through the **Schedule an Appointment** flow, the feature will open in a modal with minor design adjustments.

**Note**: The **Tire Center Locator** component should be reused with the **isModalContent** configuration to apply the UI updates when rendered inside the modal.

| Resolution | Design Reference | Scenario |
|------------|------------------|----------|
| Mobile | "docs/tires-webapp/02-design/mobile/images/Common Functionality/Change Tire Center - Modal/Change Tire Center - Modal.png" | Change Tire Center |
| Mobile | "docs/tires-webapp/02-design/mobile/images/Common Functionality/Change Tire Center - Modal/Select or Change Tire Center (Schedule an Appointment Flow) - Modal.png" | Select or Change Tire Center - Schedule an Appointment Flow |
| Desktop | "docs/tires-webapp/02-design/desktop/images/Common Functionality/Change Tire Center - Modal/Change Tire Center - Modal.png" | Change Tire Center |
| Desktop | "docs/tires-webapp/02-design/desktop/images/Common Functionality/Change Tire Center - Modal/Select or Change Tire Center (Schedule an Appointment Flow) - Modal.png" | Select or Change Tire Center - Schedule an Appointment Flow |

## Selected Vehicle/Tire Size
This section shows the selected vehicle or tire size, along with a **Change** link to update the search criteria. The section remains hidden until a search has been performed.

- Shows the **selected vehicle** or **tire size** attributes from the Redux state, located below the Header and Main Site Navigation.
- Clicking on the **Change** link will open the **Change Vehicle Modal**

## Change Vehicle Modal
Users can update the search criteria from this modal. The functionality will mirror the Tire Search on the Home Page, but the form will appear within a modal with a slightly modified design and theme.

**Note**: The Home Page Tire Search component should be reused with the **isModalContent** configuration to apply the UI updates when rendered inside the modal.

## Schedule an Appointment Modal
Users can schedule an appointment at the selected tire center for repairs, new tire installations, and additional services from this modal.

#### Scenario 1: Tire Center Already Selected
- Users must first enter their vehicle details (Year, Make, Model, Trim) before scheduling an appointment at their chosen tire center.
- If a vehicle is already selected, the form will be pre-populated with the existing vehicle information from the Redux State.
- The selected Tire Center's name is displayed with a **Change** link. Clicking this link opens the **Tire Center Locator** modal, allowing the user to select a different Tire Center and then proceed to the external Waitwhile website to complete appointment scheduling.
- If the form is submitted without changing the Tire Center, the user will be directly redirected to an external Waitwhile website to complete the appointment scheduling.

#### Scenario 2: Tire Center Not Selected
- Users must first enter their vehicle details (Year, Make, Model, Trim) and ZIP Code. The **Schedule an Appointment** button will be replaced with a **Find a Tire Center** button.
- Clicking the **Find a Tire Center** button opens the **Tire Center Locator** modal. In this flow, the **Tire Center Locator** modal's **Set as My Tire Center** button is replaced with a **Schedule an Appointment** button.
- After selecting a tire center through this modal and submitting, the user will be redirected to an external Waitwhile website to schedule the appointment.

| Resolution | Design Reference | Scenario |
|------------|------------------|----------|
| Mobile | "docs/tires-webapp/02-design/mobile/images/Common Functionality/Schedule an Appointment - Modal/Schedule an Appointment.png" | Scenario 1 |
| Desktop | "docs/tires-webapp/02-design/desktop/images/Common Functionality/Schedule an Appointment - Modal/Schedule an Appointment.png" | Scenario 1 |

## External link modal
- When users click on external links, a modal should appear with a warning that they are leaving the site and a confirmation button to continue. This ensures users are aware they are navigating away and helps prevent accidental clicks on external links.
- The modal will have two buttons: "**Cancel**" to stay on the current site and "**Visit External Link**" to continue to the external link.

| Resolution | Design Reference |
|------------|------------------|
| Mobile | "docs/tires-webapp/02-design/mobile/images/Common Functionality/External Link - Modal/External Link Modal.png" |
| Desktop | "docs/tires-webapp/02-design/desktop/images/Common Functionality/External Link - Modal/External Link Modal.png" |

## Tire Compatibility Check Modal
After searching by tire size, the Tire Compatibility Check modal appears when the user clicks **Add to Cart**, allowing them to confirm compatibility by entering their vehicle information.

The user must enter their vehicle details — Year, Make, Model, and Trim to verify if the selected tire is compatible.

#### Scenario 1: Tire is compatible
- If the tire is compatible, the modal will display a confirmation message along with **Add to Cart** and **Cancel** buttons.

#### Scenario 2: Tire is NOT compatible
- If the tire is not compatible, a warning message will appear along with **Change Vehicle** and **Continue Browsing** buttons beneath the message.
- Clicking **Change Vehicle** returns the user to the previous step to update their vehicle information.
- Clicking **Continue Browsing** closes the modal.
- The modal will also have **Add to Cart Anyway** and **Cancel** buttons.

Regardless of compatibility, the user can proceed to add the product to the cart after completing the check.

- Clicking **Add to Cart** adds the product to the cart and opens the **Added to Cart drawer**, displaying the items currently in the cart.
- Clicking **Cancel** closes the modal