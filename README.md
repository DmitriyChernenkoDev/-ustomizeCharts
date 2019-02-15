![alt tag](https://raw.github.com/PhilJay/MPChart/master/design/feature_graphic.png)

## Link to original library 

https://github.com/PhilJay/MPAndroidChart

## Classes wich where changed

1. Remove culculated offset in Y axis
file MPAndroidChart/MPChartLib/src/main/java/com/github/mikephil/charting/charts/BarLineChartBase.java
-----

change method <b>calculateOffsets</b>


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
2. Separate dates in too line 
file MPAndroidChart/MPChartLib/src/main/java/com/github/mikephil/charting/renderer/XAxisRenderer.java
-----

change method <b>drawLabels</b>

        protected void drawLabels(Canvas c, float pos, MPPointF anchor) {

        final float labelRotationAngleDegrees = mXAxis.getLabelRotationAngle();
        boolean centeringEnabled = mXAxis.isCenterAxisLabelsEnabled();

        float[] positions = new float[mXAxis.mEntryCount * 2];

        for (int i = 0; i < positions.length; i += 2) {

            // only fill x values
            if (centeringEnabled) {
                positions[i] = mXAxis.mCenteredEntries[i / 2];
            } else {
                positions[i] = mXAxis.mEntries[i / 2];
            }
        }

        mTrans.pointValuesToPixel(positions);

        for (int i = 0; i < positions.length; i += 2) {

            float x = positions[i];

            if (mViewPortHandler.isInBoundsX(x)) {

                String label = mXAxis.getValueFormatter().getFormattedValue(mXAxis.mEntries[i / 2], mXAxis);
                String[] elements = label.split(" ");

                if (mXAxis.isAvoidFirstLastClippingEnabled()) {

                    // avoid clipping of the last
                    if (i / 2 == mXAxis.mEntryCount - 1 && mXAxis.mEntryCount > 1) {
                        float width = Utils.calcTextWidth(mAxisLabelPaint, label);

                        if (width > mViewPortHandler.offsetRight() * 2
                                && x + width > mViewPortHandler.getChartWidth())
                            x -= width / 2;

                        // avoid clipping of the first
                    } else if (i == 0) {

                        float width = Utils.calcTextWidth(mAxisLabelPaint, label);
                        x += width / 2;
                    }
                }

                float y = pos+27; // float y = pos+30; for rn_Charts, 27 for other
                if(elements.length==2){
                    drawLabel(c, elements[0], x, pos, anchor, labelRotationAngleDegrees);
                    drawLabel(c, elements[1], x, y, anchor, labelRotationAngleDegrees);
                } else if(elements.length==1){
                    drawLabel(c, elements[0], x, pos, anchor, labelRotationAngleDegrees);
                }
            }
        }
    }
    
-----
3. Add indicator blocks
file MPAndroidChart/MPChartLib/src/main/java/com/github/mikephil/charting/renderer/YAxisRenderer.java
-----

change method <b>drawYLabels</b>

    protected void drawYLabels(Canvas c, float fixedPosition, float[] positions, float offset) {

        final int from = mYAxis.isDrawBottomYLabelEntryEnabled() ? 0 : 1;
        final int to = mYAxis.isDrawTopYLabelEntryEnabled()
                ? mYAxis.mEntryCount
                : (mYAxis.mEntryCount - 1);

        // draw
        for (int i = from; i < to; i++) {

            String text = mYAxis.getFormattedLabel(i);

            c.drawText(text, fixedPosition, positions[i * 2 + 1] + offset, mAxisLabelPaint);

        }

        //draw indicator block
        renderIndicatorBlock(c, fixedPosition, positions, offset);
    }
    
add new method

    private void renderIndicatorBlock(Canvas c, float positionX, float[] positions, float offset) {
        List<LimitLine> limitLines = mYAxis.getLimitLines();
        if (limitLines == null || limitLines.size() <= 0)
            return;

        float limitIndicatorBlock = limitLines.get(0).getLimit();
        int indicatorBlockColor = limitLines.get(0).getLineColor();
        Paint paint = new Paint();

        paint.setColor(indicatorBlockColor);
        paint.setStyle(Paint.Style.FILL);

        float limitFirst = mYAxis.mEntries[0];
        float gradeFirst = positions[1];

        float limitSecond = mYAxis.mEntries[1];
        float gradeSecond = positions[3];

        float limitDifferent = limitSecond - limitFirst;
        float gradeDifferent = gradeFirst - gradeSecond;

        float different = limitIndicatorBlock - limitSecond;
        float differentRangeFromZero = ((different * gradeDifferent) / limitDifferent);
        float ourRange = gradeSecond - differentRangeFromZero;

        mAxisLabelPaint.setColor(Color.WHITE);

        if (ourRange > 10 ) {
            c.drawRect(positionX - 15, ourRange - 28 + offset, positionX + 160, ourRange + 7 + offset, paint);
            c.drawText("" + limitIndicatorBlock, positionX - 11, ourRange + offset, mAxisLabelPaint);
        }
    }
