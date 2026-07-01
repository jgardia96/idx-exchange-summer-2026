I was able to access the datasets through FTP. Downloaded at least 6 months of data and the metadata pdf as well.

Key columns followed by short notes on each:

ClosePrice: Target variable, final sale price
LivingArea: Square footage totaled for living areas
BedroomsTotal: Total number of bedrooms
BathroomsTotalInteger: Total number of bathrooms
LotSizeAcres: Size of lot in acres
YearBuilt: Year when the property was built
Latitude / Longtitude: Coordinates of the property
City / CountyOrParish: Distinguishers on location
ElementarySchoolDistrict: School district where the property's located
ListPrice: Asking price from seller, DO NOT use for model
DaysOnMarket: How long the property was listed before selling
CloseDate: When the sale on the property was closed
PropertyType / PropertySubType: Columns used to filter, in reality, model uses Residential / SingleFamilyResidence

note from Aidan : "Do not use ListPrice or OriginalListPrice as features in your machine learning models, as they introduce data leakage and can artificially inflate model performance."
