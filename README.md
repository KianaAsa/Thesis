# Thesis
This Python script utilizes the pandas, matplotlib, and numpy libraries to analyze and visualize magnetic field data.
import pandas as pd
import matplotlib . pyplot as plt
import numpy as np
from scipy . interpolate import interp1d
# Load data from Excel file
file_path = ’C:/ ntnu / thesis / Results / dataset4 - new . xlsx ’
df = pd . read_excel ( file_path )
# Convert the ’Time ’ column to datetime objects and sort by time
df[’Time ’] = pd . to_datetime ( df[’Time ’], format =’%H:%M:%S’, errors =
’coerce ’)
df . sort_values ( by=’Time ’, inplace = True )
# Define the desired time range
start_time = pd . to_datetime (’15:01:00 ’, format =’%H:%M:%S’)
end_time = pd . to_datetime (’16:08:00 ’, format =’%H:%M:%S’)
# Filter data within the specified time range
filtered_df = df[( df[’Time ’] >= start_time ) & ( df[’Time ’] <=
end_time )]
# Extract magnetic data and time columns from the filtered data
magnetic_data = filtered_df [’Mag (nT)’]
time = filtered_df [’Time ’]
# Filter magnetic data less than 51000
filtered_magnetic_data = magnetic_data [ magnetic_data > 51000 ]
filtered_time = time [ magnetic_data > 51000 ]
# Create an interpolation function
interpolation_function = interp1d ( filtered_time . astype (’int64 ’) ,
filtered_magnetic_data , kind =’
linear ’, fill_value =’ extrapolate ’
)
# Define new time points for interpolation (one - minute intervals )
69
new_time_points = pd . date_range ( start = filtered_time . min () , end =
filtered_time . max () , freq =’1T ’)
# Interpolate data at new time points
interpolated_data = interpolation_function ( new_time_points . astype (
’int64 ’))
# Divide the data into 4 equal parts
num_parts = 4
part_length = len ( new_time_points ) // num_parts
average_values = []
for i in range ( num_parts ):
start_idx = i * part_length
end_idx = ( i + 1 ) * part_length
average_value = np . nanmean ( interpolated_data [ start_idx : end_idx
]) # Use nanmean to handle
NaN values
average_values . append ( average_value )
# Calculate the difference between the magnetic field first point
and the third one
difference = interpolated_data [2] - interpolated_data [0]
# Calculate the difference between the magnetic field second point
and the fourth one
difference_2_4 = interpolated_data [3] - interpolated_data [1]
# Calculate the average magnetic field values in each part
part_1_average = np . nanmean ( interpolated_data [: part_length ])
part_2_average = np . nanmean ( interpolated_data [ part_length :2*
part_length ])
part_3_average = np . nanmean ( interpolated_data [2* part_length :3*
part_length ])
part_4_average = np . nanmean ( interpolated_data [3* part_length :])
# Calculate the difference between average point in part 2 and
average point in part 4
difference_part_2_4 = part_2_average - part_4_average
# Estimate the values at the specific points where the line is
drawn
estimated_values = [ interpolated_data [0], interpolated_data [2]]
# Create subplots
fig , axs = plt . subplots (2 , 1 , figsize =( 12 , 10 ) , sharex = True )
# Plot 1: Original Magnetic Data
axs [0]. plot ( filtered_time , filtered_magnetic_data , marker =’o’,
linestyle =’-’, color =’green ’)
axs [0]. set_title (’Original Magnetic Data vs. Time ( Noisy Data
Removed )’)
axs [0]. set_ylabel (’Original Magnetic Data (nT)’)
axs [0]. grid ( True )
# Plot 2: Interpolated Magnetic Data
axs [1]. plot ( new_time_points , interpolated_data , marker =’o’,
linestyle =’-’, color =’blue ’)
70
axs [1]. set_title (’ Interpolated Magnetic Data vs. Time (1- Minute
Intervals )’)
axs [1]. set_xlabel (’Time ’)
axs [1]. set_ylabel (’ Interpolated Magnetic Data (nT)’)
axs [1]. grid ( True )
# Display the difference amount on the plot
text_x = new_time_points [0]
text_y = interpolated_data [0] + ( difference / 2 ) + 500 # Adjust
the vertical position
axs [1]. text ( text_x , text_y , f’Difference 1-3: { difference :. 2f} nT ’
, fontsize =12 , color =’red ’)
# Add a line connecting the average points
average_time_points = [ new_time_points [i * part_length ] for i in
range ( num_parts )]
axs [1]. plot ( average_time_points , average_values , marker =’o’,
linestyle =’-’, color =’red ’)
# Display the average values on the plot
axs [1]. text ( average_time_points [0], part_1_average , f’Average 1: {
part_1_average :. 2f} nT ’, fontsize
=12 , color =’orange ’)
axs [1]. text ( average_time_points [1], part_2_average , f’Average 2: {
part_2_average :. 2f} nT ’, fontsize
=12 , color =’orange ’)
axs [1]. text ( average_time_points [2], part_3_average , f’Average 3: {
part_3_average :. 2f} nT ’, fontsize
=12 , color =’orange ’)
axs [1]. text ( average_time_points [3], part_4_average , f’Average 4: {
part_4_average :. 2f} nT ’, fontsize
=12 , color =’orange ’)
# Display the difference between average point in part 2 and
average point in part 4 on the
plot
axs [1]. text ( average_time_points [1], part_2_average - 1000 , f’
Difference Part 2-4: {
difference_part_2_4 :. 2f} nT ’,
fontsize =12 , color =’black ’)
# Rotate x- axis labels for better readability
plt . xticks ( rotation =45 )
# Adjust subplot spacing
plt . tight_layout ()
# Show the plots
plt . show ()
