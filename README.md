# extract-wg Extract Water Gaps

This is a tool for ArcGIS written in Python, which will extract water gap features from an input DTM.

## Algorithm descriptions

### Algorithm 1: Input conditioning

Inputs: Raster DTM, Analysis footprint (optional, defaults to footprint of DTM), Analysis size parameter

Outputs: 	"Contour lines" - representing the analysis footprint over the input DTM (subdivided if necessary)


### Algorithm 2: Generating Contains Polygons

Inputs: Raster DTM, Contour lines, and polygon footprint to match

Outputs: "Contains polygons" - Polygon feature classes representing the contains polygons for each contour elevation 
1. Take the analysis area footprint and partition it with the contour polylines for a single height.
2. Save results to feature class according to contour height
3. For each polygon, calculate average height statistics against the source DTM.
4. Delete polygons which have an average height below the index contour height.


### Algorithm 3: Identifying Peaks and Peak Pairs

Inputs: Contains polygons, Raster DTM, prominence ratio parameter, input edge peaks (from adjacent areas)

Outputs: 	"peaks" - Point feature class representing peaks with prominence attribute, 
			"output edge peaks" - point feature class representing edge peaks for use in adjacent areas, 
			"peak pairs" - table representing pairs of peaks for analysis, paired based on threshold calculated from prominence ratio parameter

Part A Identify peak points and prominences (steps 1-5)

Part B Generate peak pairs (steps 6-12)

Part C Calculate output edge peaks (step 13)

1. Begin at the highest elevation contains polygon feature class
2. For each polygon in the class, check to see if it contains points in the peaks list
3. If polygon contains no peak points, calculate highest point in the raster DTM contained in the polygon and add to the peaks class, with the difference with the current elevation used as the initial prominence
4. Else if polygon contains one or more peaks and this represents a closed contour (does not overlap footprint edge), determine the contained peak with the highest elevation and update the prominence to be the difference between its elevation and the current contains polygon elevation.
5. Repeat at step 2 for next lowest elevation contains polygon class until all have been examined.
6. Add input edge peaks to peaks class (only include points that are along the footprint edge within a small threshold)
7. Generate a table of all size 2 combinations of peaks (lower id peak first)
8. Analyze each entry in the peaks pairs table, 
9. Discard pairs with two edge peaks
10. Determine the distance between the two peaks. For edge peaks, assume the peaks are 'distance' (input) away from the edge point in a direction orthogonal to the edge which they are associated with.
11. Calculate the cutoff distance for each peak in the pair by multiplying the prominence by the prominence ratio parameter
12. Discard pairs for which the distance between the peaks is greater than both peaks' calculated cutoff distances.
13. For peaks within the cut-off distance (calculated per-peak as in step 11) of an edge, generate an edge peak point for each sufficiently close edge paired with the distance between the edge peak point and the peak, as well as the peak's prominence. Edge peak points should be the highest ridge on the flank of a peak which crosses the edge. (probably need more detail)


### Algorithm 4: Water gap identification
Inputs: Contains polygons, peaks, peak pairs, raster DTM, streamlines, prominence threshold (might be taken care of elsewhere)?
Outputs:	"water gap candidates" - candidate water gaps identified from the algorithm with height above interpolated ridgeline as an attribute

Part A Identify mutually contained peak pairs (steps 1-4)

Part B Confirm candidate water gaps (steps 5-12)

1. Create a "working peak pairs" table as a copy of the input
2. Begin at the highest elevation contains polygon feature class
3. For each polygon in the class, check to see if it contains points in the peaks list
4. If polygon contains two or more peaks, check size 2 combinations of peaks (lower id peak first) for inclusion in the working peaks pairs table
5. Check to see if a streamline crosses between the mutually containing contour(include ephemeral streams) 
		Note: A crossing streamline is "between" the peaks if one must traverse the contour in opposite directions from the flank of each peak to arrive at the same stream.
6. If no, skip to step 11.
7. Determine straight line (interpolated ridgeline) between peak pair
8. Calculate intersection point between connecting line and streamline which crosses the mutually containing contour
9. Add intersection point to water gap candidate list
10. Calculate height between the interpolated ridgeline at the water gap point and the elevation of the terrain below. Associate this value with the water gap candidate point. Associate the trend direction of the interpolated ridgeline as a preliminary ridge trend direction.
11. Eliminate peak pair in the working peak pairs table.
12. Repeat at step 2 for next lowest elevation contains polygon class until all have been examined.


### Algorithm 5: Water gap attributes

Inputs: Raster DTM, water gap candidates

Outputs:	"water gaps" - curated, confirmed list of detected water gaps

1. Manually examine candidates, eliminate false positives, combine multiple detections (select largest height below ridgeline), note detected wind gaps on another list
2. For confirmed water gaps, calculate: 
		location, elevation of alternate divide (another tool?),
		elevation of upstream valley at divergence,
		water gap index,
		systematic name,
		Trend direction (0-180),
		Ridge trend direction,
		Length,
		Sinuosity,
    
 ## Development flow
 1. Algorithm 2
 1. Algorithm 3 without edge peak logic
 1. Algorithm 4
 1. Algorithm 1 and edge peak logic for Algorithm 3
 1. Tools for Algorithm 5 (mostly a manual process)
