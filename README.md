![alt tag](https://raw.github.com/PhilJay/MPChart/master/design/feature_graphic.png)

## Link to original library 

https://github.com/PhilJay/MPAndroidChart

## Classes wich where changed

1. Remove culculated offset in Y axis
file MPAndroidChart/MPChartLib/src/main/java/com/github/mikephil/charting/charts/BarLineChartBase.java
-----

change method calculateOffsets


            float offsetLeft = 0f, offsetRight = 0f, offsetTop = 0f, offsetBottom = 0f; //offsetBottom = 33f; for rn_Chart, 0 for other

            calculateLegendOffsets(mOffsetsBuffer);

            offsetLeft += mOffsetsBuffer.left;
            offsetTop += mOffsetsBuffer.top;
            offsetRight += mOffsetsBuffer.right;
            offsetBottom += mOffsetsBuffer.bottom;

            // offsets for y-labels
            if (mAxisLeft.needsOffset()) {
            //          offsetLeft += mAxisLeft.getRequiredWidthSpace(mAxisRendererLeft
            //          .getPaintAxisLabels());
                offsetLeft = 50f;
            }

            if (mAxisRight.needsOffset()) {
            //           offsetRight += mAxisRight.getRequiredWidthSpace(mAxisRendererRight
            //           .getPaintAxisLabels());
                offsetRight += 160f;
            }
            
-----
