# bangalore-hotel-data-analysis
R data analysis project ranking Bangalore hotels by value using hotel scoring, visualizations, t-tests, regression, and ANOVA.

## Goal
Main goal of this project is to determine which hotels in Bangalore India is the best overall value based on guest rating, review count, price, distance from the city center and room size.

## Motivation 
I love traveling and I been to 21 countries plus cool places like Guam and Puerto Rico. Travelers usually care about more than just price.  A good hotel should have good balance between cost, location, reviews, roomsize and overall quality. This projects creates a hotel scoring system to compare hotels more systematically. 

## Dataset
The dataset used in this proejct was the Bangalore Hotel Booking Dataset (2026) from Kaggle. 
Dataset link: https://www.kaggle.com/datasets/samhoon/bangalore-hotel-booking-dataset-2026
The dataset contains 500 hotel listings from Bangalore, India. 

## Variables
hotel_name
area
star_rating
room_type 
price_per_night_inr
guest_rating
review_count
room_size_sqft
distance_from_city

## Dervied Variables## Derived Variables

To compare hotels more fairly, I created several derived variables from the original dataset. These variables helped standardize different hotel features and build an overall hotel value score.

### Normalized Factor Scores

Because the variables were measured on different scales, I converted them into normalized scores between 0 and 1.

- `rating_score`: normalized version of `guest_rating`
- `review_score`: normalized log-transformed version of `review_count`
- `room_size_score`: normalized version of `room_size_sqft`
- `price_score`: normalized price score where lower prices receive higher scores
- `distance_score`: normalized distance score where hotels closer to the city center receive higher scores

```r
rating_score = guest_rating / max(guest_rating, na.rm = TRUE)

review_score = log(review_count + 1) / max(log(review_count + 1), na.rm = TRUE)

room_size_score = room_size_sqft / max(room_size_sqft, na.rm = TRUE)

price_score = 1 - (price_per_night_inr / max(price_per_night_inr, na.rm = TRUE))

distance_score = 1 - (distance_from_city_center_km / max(distance_from_city_center_km, na.rm = TRUE))
```

### Hotel Score

I created an overall `hotel_score` by giving equal weight to five factors: guest rating, review count, price, distance from the city center, and room size.

```r
hotel_score =
  0.20 * rating_score +
  0.20 * review_score +
  0.20 * price_score +
  0.20 * distance_score +
  0.20 * room_size_score
```

A higher `hotel_score` means the hotel has a stronger overall balance of high guest ratings, many reviews, lower price, larger room size, and closer distance to the city center.

### Hotel Score Group

I created `top_20_group` to separate the 20 hotels with the highest hotel scores from the rest of the dataset.

- `Best 20 Hotel Score`
- `Out of Top 20`

This allowed me to compare the highest-ranked hotels against all other hotels.

### Price Group

I created `price_groups` by comparing each hotel’s price per night to the median price.

- `Cheap Hotel`: hotels at or below the median price
- `Expensive Hotel`: hotels above the median price

This allowed me to compare cheap and expensive hotels using boxplots and Welch two-sample t-tests.

## Research Questions
1. Which hotels in Bangalore provide the best overall value?
2. Do cheap and expensive hotels differ in guest rating, room size, distance, star rating, and price?
3. Do the top 20 hotel-score hotels differ from the rest of the hotels?
4. Which variable best predicts hotel price per night?
5. Is star rating significantly related to price per night?

## Methods
All analysis was conducted in R using `tidyverse`, `dplyr`, `ggplot2`, `janitor`, `broom`, and base R statistical functions.

The analysis used:
- Data cleaning and variable name standardization
- Derived variable creation
- Hotel value score construction
- Exploratory data visualization
- Boxplots for group comparisons
- Two-sample t-tests
- Simple linear regression
- Multiple linear regression
- Adjusted R-squared model comparison
- ANOVA

## Statistical Analysis

### Welch Two-Sample T-Tests

I used Welch two-sample t-tests to compare the means of different hotel groups. Welch two-sample t-tests were appropriate because the comparisons involved two independent groups, such as cheap vs. expensive hotels or Best 20 Hotel Score hotels vs. hotels outside the top 20. The tests helped determine whether the differences seen in the boxplots were statistically significant.

The main grouping variables were:

- `price_groups`: compares `Cheap Hotel` and `Expensive Hotel`
- `top_20_group`: compares `Best 20 Hotel Score` and `Out of Top 20`

The t-tests compared group means for guest rating, room size, distance from the city center, star rating, and price per night.

### T-Test Results Summary

| Test | P-value | 95% Confidence Interval | Result |
|---|---:|---:|---|
| Guest rating by price group | 0.5619 | (-0.0677, 0.1245) | Not statistically significant |
| Guest rating by hotel-score group | 1.871e-11 | (0.5898, 0.8544) | Statistically significant |
| Room size by price group | 0.592 | (-30.4843, 17.4123) | Not statistically significant |
| Room size by hotel-score group | 0.05069 | (-0.1872, 113.8455) | Not statistically significant at the 5% level |
| Distance from city center by price group | 0.04363 | (0.0328, 2.2516) | Statistically significant |
| Distance from city center by hotel-score group | 0.0002556 | (-5.9871, -2.1049) | Statistically significant |
| Star rating by price group | < 2.2e-16 | (-1.4536, -1.2904) | Statistically significant |
| Star rating by hotel-score group | 2.645e-06 | (-0.9474, -0.4693) | Statistically significant |
| Price per night by price group | < 2.2e-16 | (-11239.257, -9782.463) | Statistically significant |
| Price per night by hotel-score group | 5.184e-15 | (-6634.920, -4618.621) | Statistically significant |

### Interpretation of T-Test Results

The t-test results showed that cheap and expensive hotels did not have a statistically significant difference in average guest rating or average room size. This means that, in Bangalore India, cheaper hotels were not necessarily rated lower by guests and did not have significantly smaller rooms than expensive hotels.

However, cheap and expensive hotels did show statistically significant differences in distance from the city center, star rating, and price per night. Cheap hotels were farther from the city center on average, had lower star ratings, and had lower prices per night than expensive hotels.

The comparison between the Best 20 Hotel Score group and the hotels outside the top 20 showed stronger differences. The Best 20 Hotel Score group had significantly higher guest ratings, significantly lower prices, and was significantly closer to the city center. These results suggest that the top-ranked hotels were not simply the most expensive hotels. Instead, they provided stronger overall value by balancing high guest ratings, lower prices, and better location.

The Best 20 Hotel Score group did not have a statistically significant difference in room size at the 5% level. Although the top 20 hotels had a larger average room size, the p-value was slightly greater than 0.05, so there was not enough evidence to conclude that the difference in room size was statistically significant.

One important limitation is that some of these results should be interpreted carefully because the `hotel_score` was created using guest rating, review count, price, distance, and room size. Therefore, comparisons involving the Best 20 Hotel Score group may partly reflect how the score was constructed.

## Visualizations
