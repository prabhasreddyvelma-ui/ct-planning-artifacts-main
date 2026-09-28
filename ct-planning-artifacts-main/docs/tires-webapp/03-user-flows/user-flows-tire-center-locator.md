# Tire Center Locator (Standalone Page)
The standalone Tire Center Selection page is displayed only the first time a user selects a tire center. For subsequent changes—such as updating the tire center or selecting one during the appointment scheduling flow—the interface opens in a modal window.

**Full Page Design**:

| Resolution | Design Reference |
|------------|------------------|
| Mobile | "docs/tires-webapp/02-design/mobile/images/Tire Center Locator/Tire Center Locator.png" |
| Desktop | "docs/tires-webapp/02-design/desktop/images/Tire Center Locator/Tire Center Locator.png" |

This interface lets users select a tire center for shipping and installation. It includes a unified search, allowing users to search by city, state, or ZIP code to locate nearby tire centers.

#### Scenario 1: Tire Centers Found Near the User
- A list of nearby tire centers is displayed, including details and a button to select a tire center.
- Tire centers are also plotted on a Bing Map, shown alongside the list.
- Clicking a marker on the map highlights the corresponding tire center in the list and displays its details.

#### Scenario 2: No Tire Centers Found
- If no tire centers are found, the list is replaced with the message stating "We were unable to find a Costco warehouse near your location. Try searching again."
- Any markers from a previous search on the Bing Map are cleared.

Once a tire center is selected, the header link updates to show the chosen tire center name.
The selection is saved in Redux state and persisted in localStorage to maintain session continuity.

**Note**: Developers must be able to pass a configuration, such as **isModalContent**, to modify the UI when the component is rendered inside a modal.