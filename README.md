# Literature Review

Alexander, Christopher, Sara Ishikawa, and Murray Silverstein. A pattern language: Towns, buildings, construction. New York, NY: Oxford Univ. Press, 1977. 

CASTELLS, Manuel. The Informational City: Information Technology, economic, restructuring, and the urban-regional process. Oxford, Oxfordshire: Blackwell, 1989.

Kelly-Holmes, Kelly. Advertising as multilingual communication. New York, NY: Palgrave Macmillan, 2005. 

Nasar, Jack L. The evaluative image of the city. Thousand Oaks, CA: Sage Publications, 1998. 

Phillipson, Robert. Linguistic imperialism Robert Phillipson. Oxford, Oxfordshire: Oxford University Press, 1992. 

Portella, Adriana. Visual pollution : advertising, signage and environmental quality. Burlington, Vermont: Ashgate, 2014. 


# 01D Thesis Abstract

Today, many people discover food through digital platforms instead of walking around a neighborhood. Apps like Uber Eats and Google Maps do not simply list restaurants; they organize food into categories, icons, and search results that shape how users navigate and choose.

I am studying how Uber Eats categorizes food, focusing on several noodle-based dishes as case studies. While some dishes appear prominently with their own labels or icons, others are harder to find unless users type the exact name.

I want to understand how this system of categorization influences what becomes easily discoverable and what requires extra effort to locate.

By examining these differences, this project aims to better understand how digital classification structures the way cultural food identities are presented and experienced online.


# 01E Thesis Introduction

In many cities today, food is discovered less through walking and more through digital platforms. Instead of encountering restaurants by chance, people scroll through curated lists, icons, and search results on apps like Uber Eats. These platforms do more than display options. They organize food into structured categories that shape how users browse and what they notice first.

This organization is not random. Certain dishes appear prominently, sometimes with their own visual category or quick-access icon. Others exist within broader groupings, or only become visible when users type their exact names into the search bar. As a result, the experience of discovering food is influenced not only by taste or geography, but by how the platform structures its classification system.

In this thesis examines Uber Eats as a system of categorization rather than simply an interface. By focusing on several noodle-based dishes as case studies, the project explores how different foods occupy different positions within the platform’s structure. Some are treated as distinct categories, while others are grouped under broader national or generic labels.

The goal of this research is not to redesign the platform. Instead, it seeks to understand how digital classification shapes food visibility and, in turn, how cultural food identities are presented within a structured digital environment.

By analyzing these patterns, the project aims to reveal how the organization of categories influences discovery, and how that organization subtly shapes users’ perception of what kinds of food are central, familiar, or peripheral.

# 02A Concepts and Sketches

## concept sketches for food taxonomy and platform visibility

three visualization concepts exploring how platform level food classification reshapes spatial and semantic visibility with sample data.

Dataset:
- OpenStreetMap cuisine data (1,171 cuisine entries)
- 131 unique cuisine categories
- Uber entry-level taxonomy (40 categories)

## 1. Distorted Cartogram – Platform Visibility Ratio

![Distorted Cartogram](./concepts/1.png)

This visualization explores how platform visibility reshapes geography.

Question:
If Uber’s entry-level taxonomy determined spatial prominence, which neighborhoods would expand or contract?

Method:
Neighborhood areas are scaled by a Visibility Ratio:

Visibility Ratio = mapped categories / total OSM categories

## 2. Platform Lens – Same City, Different Classification

![Platform Lens](./concepts/2.png)

This visualization compares how the same restaurants appear under different classification systems.

Question:
How does the city change when the taxonomy layer changes?

Method:
- OSM view retains 131 cuisine categories
- Uber entry view collapses categories to 40
- Same restaurant locations, different classification output

## 3. Unrepresented Cuisine Map – What Exists but Is Not Indexed

![Unrepresented Cuisine Map](./concepts/3.png)

This map highlights cuisine categories present in OSM but not represented in Uber’s entry-level taxonomy.

Question:
Which cuisines exist in the city but are absent from entry-level platform classification?

Method:
Cuisine entries without a corresponding Uber category are marked as “unrepresented.”
Only these categories are visualized.


# 02B Design Mockup

## 1. NYC Spatial Restructuring

![NYC Spatial Restructuring](./concepts/mock1.png)

## 1. NYC Multi-Label Cuisine Tag Network

![NYC Multi-Label Cuisine Tag Network](./concepts/mock2.png)
