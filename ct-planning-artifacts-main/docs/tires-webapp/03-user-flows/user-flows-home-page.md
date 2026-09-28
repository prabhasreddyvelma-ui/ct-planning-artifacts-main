# Home Page

**Full Page Design**:

| Resolution | Design Reference |
|------------|------------------|
| Mobile | "docs/tires-webapp/02-design/mobile/images/Home Page/Home Page.png" |
| Desktop | "docs/tires-webapp/02-design/desktop/images/Home Page/Home Page.png" |

## Hero Banner with Tire Search Form
The Home Page will feature a hero banner containing a Tire Search form with multiple search options.

### License Plate:
Users can search for tires by entering their vehicle's **License Plate Number** and choosing their **State**. This option is limited to users in the US and is not available to users in Canada.

#### Scenario 1: Partial Search, entering only part of the license plate (Multiple vehicles found)
If the search returns multiple vehicle matches, the user will see a message stating "We found multiple vehicles for this license plate" along with a list of the matched vehicles. The user must select the correct vehicle and confirm, after which they will be redirected to the **Size Selection Page**.

#### Scenario 2: Vehicle found, But Trim Required
In this scenario, when a matching vehicle is found but requires additional information (Trim), the user must enter the Trim details and confirm before proceeding to the **Size Selection Page**.

#### Scenario 3: No Vehicle found
If the search returns no matches, the user will see a message stating, "We were unable to match your license plate to a vehicle. Please try again or search by Vehicle, Size, or Item #."

#### Scenario 4: Vehicle found
User will be redirected to **Size Selection Page**.

| Resolution | Design Reference | Scenario |
|------------|------------------|----------|
| Mobile | "docs/tires-webapp/02-design/mobile/images/Home Page/Tire Search - Search options/Search by License Plate/Search by License Plate (No Zip Code).png" | Search by License Plate - No Zip Code |
| Mobile | "docs/tires-webapp/02-design/mobile/images/Home Page/Tire Search - Search options/Search by License Plate/Search by License Plate (With Zip Code).png" | Search by License Plate - With Zip Code |
| Mobile | "docs/tires-webapp/02-design/mobile/images/Home Page/Tire Search - Search options/Search by License Plate/Search by License Plate - Partial License.png" | Scenario 1 |
| Mobile | "docs/tires-webapp/02-design/mobile/images/Home Page/Tire Search - Search options/Search by License Plate/Search by License Plate - License Plate Found.png" | Scenario 2 |
| Desktop | "docs/tires-webapp/02-design/desktop/images/Home Page/Tire Search - Search options/Search by License Plate/Search by License Plate (No Zip Code).png" | Search by License Plate - No Zip Code |
| Desktop | "docs/tires-webapp/02-design/desktop/images/Home Page/Tire Search - Search options/Search by License Plate/Search by License Plate (With Zip Code).png" | Search by License Plate - With Zip Code |
| Desktop | "docs/tires-webapp/02-design/desktop/images/Home Page/Tire Search - Search options/Search by License Plate/Search by License Plate - Partial License.png" | Scenario 1 |
| Desktop | "docs/tires-webapp/02-design/desktop/images/Home Page/Tire Search - Search options/Search by License Plate/Search by License Plate - License Plate Found.png" | Scenario 2 |

### Vehicle:
Users can search for tires by selecting their vehicle's **Year**, **Make**, **Model**, and **Trim**, and will then be redirected to the **Size Selection Page**.

| Resolution | Design Reference | Scenario |
|------------|------------------|----------|
| Mobile | "docs/tires-webapp/02-design/mobile/images/Home Page/Tire Search - Search options/Search by Vehicle/Search by Vehicle (No Zip Code).png" | Search by Vehicle - No Zip Code |
| Mobile | "docs/tires-webapp/02-design/mobile/images/Home Page/Tire Search - Search options/Search by Vehicle/Search by Vehicle (With Zip Code).png" | Search by Vehicle - With Zip Code |
| Desktop | "docs/tires-webapp/02-design/desktop/images/Home Page/Tire Search - Search options/Search by Vehicle/Search by Vehicle (No Zip Code).png" | Search by Vehicle - No Zip Code |
| Desktop | "docs/tires-webapp/02-design/desktop/images/Home Page/Tire Search - Search options/Search by Vehicle/Search by Vehicle (With Zip Code).png" | Search by Vehicle - With Zip Code |

### Tire Size:
Users can search for tires by selecting **Width**, **Aspect Ratio**, and **Diameter**. When using the tire size search, they can choose either the same size for front and rear tires or different sizes for each.

#### Scenario 1: User selects same front and rear tire sizes
Upon submitting the search, users will be redirected to the **Products Listing Page**.

#### Scenario 2: User selects different rear tire sizes
Upon submitting the search, users will be redirected to the **Products Listing Page**.

| Resolution | Design Reference | Scenario |
|------------|------------------|----------|
| Mobile | "docs/tires-webapp/02-design/mobile/images/Home Page/Tire Search - Search options/Search by Size/Search by Size (No Zip Code) - Same Front & Rear Tires Size (Toggle 1).png" | Scenario 1 - No Zip Code |
| Mobile | "docs/tires-webapp/02-design/mobile/images/Home Page/Tire Search - Search options/Search by Size/Search by Size (With Zip Code) - Same Front & Rear Tires Size (Toggle 1).png" | Scenario 1 - With Zip Code |
| Mobile | "docs/tires-webapp/02-design/mobile/images/Home Page/Tire Search - Search options/Search by Size/Search by Size (No Zip Code) - Different Front & Rear Tires Size (Toggle 2).png" | Scenario 2 - No Zip Code |
| Mobile | "docs/tires-webapp/02-design/mobile/images/Home Page/Tire Search - Search options/Search by Size/Search by Size (With Zip Code) - Different Front & Rear Tires Size (Toggle 2).png" | Scenario 2 - With Zip Code |
| Desktop | "docs/tires-webapp/02-design/desktop/images/Home Page/Tire Search - Search options/Search by Size/Search by Size (No Zip Code) - Same Front & Rear Tires Size (Toggle 1).png" | Scenario 1 - No Zip Code |
| Desktop | "docs/tires-webapp/02-design/desktop/images/Home Page/Tire Search - Search options/Search by Size/Search by Size (With Zip Code) - Same Front & Rear Tires Size (Toggle 1).png" | Scenario 1 - With Zip Code |
| Desktop | "docs/tires-webapp/02-design/desktop/images/Home Page/Tire Search - Search options/Search by Size/Search by Size (No Zip Code) - Different Front & Rear Tires Size (Toggle 2).png" | Scenario 2 - No Zip Code |
| Desktop | "docs/tires-webapp/02-design/desktop/images/Home Page/Tire Search - Search options/Search by Size/Search by Size (With Zip Code) - Different Front & Rear Tires Size (Toggle 2).png" | Scenario 2 - With Zip Code |

### Item No./Part No.:
User can search for tires by entering **Item No.** Or **Part No.**.

#### Scenario 1: User enters a valid Item Number or Part Number
Upon submitting the search, users will be redirected to the **Product Details Page**.

#### Scenario 2: User enters an invalid Part Number
When a user enters an invalid Part Number and submits the form, a custom error message stating "Part number does not exist" will be displayed.

**Note**: 

- For all search types, the form will include an additional **ZIP Code** field if no Tire Center has been selected. Submitting the form opens the **Tire Center Locator** modal, allowing the user to choose a Tire Center before proceeding to the appropriate page based on the search type.
- All form field errors in the Tire Search form—across all search types—including required fields, should follow the custom error message style specified in the design reference, instead of using the default HTML5 error presentation. For example, the dropdown or text input will show a red border, with the error message displayed directly below the field. The background color of the dropdown or text input should remain unchanged.
- Search criteria should be saved in the Redux State and persisted in localStorage to maintain session continuity.
- This form can be accessed via a modal (**Change Vehicle Modal**) when the user wants to change the search criteria from subsequent pages. The same component must be reused. Developers must be able to pass a configuration, such as **isModalContent**, to modify the UI when the component is rendered inside a modal.

| Resolution | Design Reference | Scenario |
|------------|------------------|----------|
| Mobile | "docs/tires-webapp/02-design/mobile/images/Home Page/Tire Search - Search options/Search by Item Number or Part Number/Search by Item Number (No Zip Code).png" | Scenario 1 - Search by Item Number - No Zip Code |
| Mobile | "docs/tires-webapp/02-design/mobile/images/Home Page/Tire Search - Search options/Search by Item Number or Part Number/Search by Item Number (With Zip Code).png" | Scenario 1 - Search by Item Number - With Zip Code |
| Mobile | "docs/tires-webapp/02-design/mobile/images/Home Page/Tire Search - Search options/Search by Item Number or Part Number/Search by Part Number (No Zip Code) - Error State.png" | Scenario 2 - Search by Part Number - No Zip Code - Invalid Part Number Error |
| Mobile | "docs/tires-webapp/02-design/mobile/images/Home Page/Tire Search - Search options/Search by Item Number or Part Number/Search by Part Number (With Zip Code) - Error State.png" | Scenario 2 - Search by Part Number - With Zip Code - Invalid Part Number Error |
| Desktop | "docs/tires-webapp/02-design/desktop/images/Home Page/Tire Search - Search options/Search by Item Number or Part Number/Search by Item Number (No Zip Code).png" | Scenario 1 - Search by Item Number - No Zip Code |
| Desktop | "docs/tires-webapp/02-design/desktop/images/Home Page/Tire Search - Search options/Search by Item Number or Part Number/Search by Item Number (With Zip Code).png" | Scenario 1 - Search by Item Number - With Zip Code |
| Desktop | "docs/tires-webapp/02-design/desktop/images/Home Page/Tire Search - Search options/Search by Item Number or Part Number/Search by Part Number (No Zip Code).png" | Scenario 1 - Search by Part Number - No Zip Code |
| Desktop | "docs/tires-webapp/02-design/desktop/images/Home Page/Tire Search - Search options/Search by Item Number or Part Number/Search by Part Number (With Zip Code).png" | Scenario 1 - Search by Part Number - With Zip Code |
| Desktop | "docs/tires-webapp/02-design/desktop/images/Home Page/Tire Search - Search options/Error States/Form Field - Error State.png" | Scenario 2 - Search by Part Number - With Zip Code - Invalid Part Number Error |

## Deals Carousel
This section features a slider for deals and discounts, with play and pause buttons.

| Resolution | Design Reference |
|------------|------------------|
| Mobile | "docs/tires-webapp/02-design/mobile/images/Home Page/Tires Deals - Slider/Tires Deals - Slider with Play and Pause buttons.png" |
| Desktop | "docs/tires-webapp/02-design/desktop/images/Home Page/Tires Deals - Slider/Tires Deals - Slider with Play and Pause buttons.png" |

**Use the below slide images**:

| Slide Index | Slide Image |
|-------------|-------------|
| 1 | "docs/tires-webapp/02-design/desktop/images/Home Page/Tires Deals - Slider/Slides/Slide 1 - Michelin.png" |
| 2 | "docs/tires-webapp/02-design/desktop/images/Home Page/Tires Deals - Slider/Slides/Slide 2 - Firestone.png" |
| 3 | "docs/tires-webapp/02-design/desktop/images/Home Page/Tires Deals - Slider/Slides/Slide 3 - Bridgestone.png" |

## Why buy tires at Costco?
Blog section - Highlights the key benefits of purchasing tires at Costco and provides the user with an option to schedule an appointment.

| Resolution | Design Reference |
|------------|------------------|
| Mobile | "docs/tires-webapp/02-design/mobile/images/Home Page/Why Buy Tires at Costco/Why Buy Tires at Costco.png" |
| Desktop | "docs/tires-webapp/02-design/desktop/images/Home Page/Why Buy Tires at Costco/Why Buy Tires at Costco.png" |

## Tire Information
This section features a slider showcasing tire information slides. Clicking a slide will redirect the user to an external link.

| Resolution | Design Reference |
|------------|------------------|
| Mobile | "docs/tires-webapp/02-design/mobile/images/Home Page/Tire Information - Slider/Tire Information - Slider with Play and Pause buttons.png" |
| Desktop | "docs/tires-webapp/02-design/desktop/images/Home Page/Tire Information - Slider/Tire Information - Slider with Play and Pause buttons.png" |

**Use the below slide images**:

| Slide Index | Slide Image |
|-------------|-------------|
| 1 | "docs/tires-webapp/02-design/desktop/images/Home Page/Tire Information - Slider/Slides/Slide 1 - How To Buy Tires.png" |
| 2 | "docs/tires-webapp/02-design/desktop/images/Home Page/Tire Information - Slider/Slides/Slide 2 - When To Buy Tires.png" |
| 3 | "docs/tires-webapp/02-design/desktop/images/Home Page/Tire Information - Slider/Slides/Slide 3 - Tire Care.png" |
| 4 | "docs/tires-webapp/02-design/desktop/images/Home Page/Tire Information - Slider/Slides/Slide 4 - Frequently Asked Questions.png" |
| 5 | "docs/tires-webapp/02-design/desktop/images/Home Page/Tire Information - Slider/Slides/Slide 5 - Tire Glossary.png" |
| 6 | "docs/tires-webapp/02-design/desktop/images/Home Page/Tire Information - Slider/Slides/Slide 6 - Warranty Information.png" |
| 7 | "docs/tires-webapp/02-design/desktop/images/Home Page/Tire Information - Slider/Slides/Slide 7 - The Costco Advantage.png" |
| 8 | "docs/tires-webapp/02-design/desktop/images/Home Page/Tire Information - Slider/Slides/Slide 8 - Legal Disclaimers.png" |

## Blog Section
Displays an article for "Best place to work" with a full-wdth warehouse image

| Resolution | Design Reference |
|------------|------------------|
| Mobile | "docs/tires-webapp/02-design/mobile/images/Home Page/The Best Place to Buy Tires/The Best Place to Buy Tires.png" |
| Desktop | "docs/tires-webapp/02-design/desktop/images/Home Page/The Best Place to Buy Tires/The Best Place to Buy Tires.png" |