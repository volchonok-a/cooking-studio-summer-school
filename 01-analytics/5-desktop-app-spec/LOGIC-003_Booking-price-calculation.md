# Booking Price Calculation

## Overview
Describes the real-time calculation of the total cost displayed on `SCR-004`.

## Formula
$$\text{Total} = (\text{Base Price} \times \text{Places}) + \text{Rental Cost} - \text{Loyalty Discount}$$

## Business Rules
- **Base Price:** Fetched per slot.
- **Rental Cost:** Flat fee (+500 ₽) if equipment rental is selected.
- **Loyalty Discount:** Applied automatically (5% off) if `visit_count > 5` in user profile.

## Links
- FR-11, FR-14.