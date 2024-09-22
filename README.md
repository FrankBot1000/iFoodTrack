# iFoodTrack
iFoodTrack is a macOS App built using Swift and AppKit that integrates the USDA FoodCentral database. You can look up food details, save favorites, maintain a food diary and track nutrient and food count totals in charts. 

iFoodTrack is organized with a left navigation bar, a middle detail view, for charts and lists, and a right extended details view describing data averages and trends. 

You can save/open food diary and favorites files, and export data in csv format. Nutrient data can be tracked over time in pie charts, bar charts and stacked-bar charts, with related-foods listed below charts.

# Technologies Used								
* Swift Programming language
* AppKit framework
* Core Data
* Core Graphics
* AutoLayout
* Unit Testing
* Documentation (DocC)
* Git Version Control
* Security/Assembly Language
* File Management, Keychain Files

# iFoodTrack Animation
<figure>
	<div>
		<video width="500" controls poster="images/screenshots/02a iFTmacOS main window TimeChart light.png" muted preload="auto">
			<source src="videos/iFoodTrackmacOS_compressed.mp4" type="video/mp4">
			<!-- For non-HTML5 browsers: -->
			Your browser doesn't support the video tag. Click <a href=http://www.firefox.com>here</a> 
			to download the Firefox browser for your operating system.
		</video>
	</div>
</figure>

# An AppKit Project using Programmatic-UI...
Rather than using Xcode's Interface Builder and Cocoa Bindings for building and connecting Views, iFoodTrack views and window components were built using only Programmatic-UI. By doing everything programmatically, it allowed me to better manage an organized and structured code base under git source control. It also gave me a deeper appreciation for how things like UITableView and window components are built. Another challenge was building a navigation system using protocols and delegates to communicate data changes for different parts of the App.

In order to achieve backward compatibility with older macOS versions, charts for displaying nutrient and food count data were built from the ground up. This included the updating and formatting of x- and y- axis values and labels, a y-axis average-value dashed-line, and a mouse-tracking vertical line for selecting individual dates in bar charts to show updated daily data.
<br></br>

## Sample Code:
### Plotting daily nutrient totals in time-bar charts:
```swift
/// Will draw nutrient-specific nutrient total value per date bar of bar graph.
    func drawBarGraphDataSegments(_ cgContext: CGContext?) {
        let lineWidth    = 2.0                              // initialize with zero value
        var y            = lineWidth - 1.0 + xLabelHeight   // start half way up width of axis line
        
        // total the value of all bars ( = "segments") by using reduce to sum them
        guard let maxValue  = segments.map({Double($0.value)}).max() else { return }
        var maxBarHeight    = maxValue > 0 ? maxValue : 100.0   // protects against dividing by zero.
        
        // Check if drv100Percent value is greater than maxValue, then set max to drv100Percent value
        maxBarHeight        = drv100PercentValue >= maxValue ? drv100PercentValue : maxValue
        
        let roundedRect     = FBRoundedRect(radius: 2.5, corners: [.bottomLeft, .bottomRight])
        // To round the top of bar, use bottomLeft and bottomRight, since drawing starts from bottom to top
        
        // Loop through the values array:
        for (index, segment) in segments.enumerated() {
            let i: CGFloat  = CGFloat(index)
            let x: CGFloat  = i * (barWidth + spacing) + offset + yLabelHeight
            let value       = segment.value > 0 ? segment.value : 0 // set to zero if value is negative.
            
            let barHeight   = (value/maxBarHeight * (viewHeight - xLabelHeight))
            if barHeight == 0 { continue }  // If barHeight was zero, then skip drawing the bar! Prevents drawing artifacts on X-Axis.
            
            let bar         = CGRect(x: x, y: y, width: barWidth, height: barHeight)
            var path        = CGPath(rect: .zero, transform: nil)
            path            = roundedRect.path(in: bar) ?? CGPath(roundedRect: bar, cornerWidth: 2.5, cornerHeight: 2.5, transform: .none)
            
            cgContext?.setFillColor(self.nutrientColor.cgColor)
            cgContext?.addPath(path)
            cgContext?.fillPath()
            
            y = lineWidth - 1.0 + xLabelHeight     // reset y
        }
    }
```
<br>
</br>

### A test for validating nutrient value calculations:
```swift
/// Test will test Food Nutrient DRV percent calculation using DRV nutrient defaults. The result should yield 100% for all DRV Nutrient calculations.
    func testCalculateNutValues_recalculateDRVPercent() {
        // Arrange
        let other6NutItemIndexes: [kNutrientNameIndex] = FBDRVSettingsHelper.shared.nutNameIndexes
        
        // MARK: Set up NutIDs:
        // topNutIDs, midNutIDs, and bottomNutIDs split 'other6NutIems' into subarray item pairs:
        var otherNutIDs: (topPair:[kNutrientID], midPair:[kNutrientID], bottomPair:[kNutrientID]) {
            let topPair = other6NutItemIndexes[0...1].map({$0.rawValue})
            let midPair = other6NutItemIndexes[2...3].map({$0.rawValue})
            let btmPair = other6NutItemIndexes[4...5].map({$0.rawValue})
            let NutPairs = [topPair, midPair, btmPair]
            let NutIDPairs = NutPairs.map{$0.map{kNutrientID.allNutrientIDs[$0]}}
            return (NutIDPairs[0], NutIDPairs[1], NutIDPairs[2])
        }
        
        var topNutIDs: [kNutrientID]    { otherNutIDs.topPair } // recompute .topPair
        var midNutIDs: [kNutrientID]    { otherNutIDs.midPair } // recompute .midPair
        var bottomNutIDs: [kNutrientID] { otherNutIDs.bottomPair } // recompute .bottomPair
        
        let otherNutIDsArrays   = [topNutIDs, midNutIDs, bottomNutIDs]
        
        // MARK: Create DRV Food (a token food):
        let nutIDs      = kNutrientID.allCases.map{$0.rawValue}
        let nutNames    = kNutrientName.allCases.map{$0.rawValue}
        let nutNumbers  = kNutrientNameIndex.allNutrientIndexes
        let nutModifers = kNutrientNameIndex.allUnitModifiers
        let nutDRV      = NutrientDRV()

        var nutrients: [FDRNutrient] = []

        // Iterate through all nutrients, based on nutrient name case number (index) in kNutrientNameIndex enum:
        for n in nutNumbers {
            nutrients.append(FDRNutrient(amount: nutDRV.nutrientDRVDict[kNutrientNameIndex(rawValue: n)!], nutrient: FDRNutInfo(id: nutIDs[n], number: "0", name: nutNames[n], rank: 0, unitName: nutModifers[n])))
        }
        
        
        var drvFood  = FDRFood(nutrients: nutrients)
        var drvAbsAmountLabelsArray: [FBNutAmountsAndLabels] = []
        let graphVC = FBGraphVC()
        
//        let expectedOtherNutIDsArrays: [FBNutAmountsAndLabels] = [
//        FBNutAmountsAndLabels(nutAmounts: (drvAmounts: [100, 100], absAmounts: [20, 28]), nutLabels: (drvLabels: ["%", "%"], absLabels: ["g", "g"])),
//        FBNutAmountsAndLabels(nutAmounts: (drvAmounts: [100, 100], absAmounts: [90, 300]), nutLabels: (drvLabels: ["%", "%"], absLabels: ["g", "mg"])),
//        FBNutAmountsAndLabels(nutAmounts: (drvAmounts: [100, 100], absAmounts: [1300, 2300]), nutLabels: (drvLabels: ["%", "%"], absLabels: ["mg", "mg"]))
//        ]
        
        // Act
        // Calculate selected Food amounts:
        for otherNutIDsArray in otherNutIDsArrays {
            guard let nutAmountsAndLabels = drvFood.calculateNutValues(nutrientIDs: otherNutIDsArray) else { return }
            drvAbsAmountLabelsArray.append(nutAmountsAndLabels)
        }
        
        let drvOtherNutIdsAmounts = drvAbsAmountLabelsArray.map{$0.nutAmounts}.map{$0.drvAmounts}
        let expectedDRVOtherNutIdsAmounts = [[100.0, 100.0], [100.0, 100.0], [100.0, 100.0]]
        
        // Assert
        XCTAssert(drvOtherNutIdsAmounts == expectedDRVOtherNutIdsAmounts)
        
    }
```
<br>
</br>


### A test to confirm conversion of double values to scientific notation, for display in chart y-axis labels:

```swift
    func testConvertDoubleToSciNotationForGraph() {
        // Given
        let segmentValues = [0,
                            1,
                             1.2,
                             1.23,
                             1.2345678,
                            1234,
                             1234.5678,
                             0.12345678,
                             0.012345678,
                             0.0012345678]

        let numFormatter = NumberFormatter()
        numFormatter.maximumFractionDigits = 1
        numFormatter.numberStyle = .scientific

        for segmentValue in segmentValues {
            let myNum_sciString     = numFormatter.string(for: segmentValue) ?? ""
            
            // When
            let myNum_beforeDecimal   = myNum_sciString.prefix(3)
            
            // Will include "E" if was a whole number or zero, so only keep first character:
            var myNum_coefficientPart = myNum_beforeDecimal // NB: will contain "E" when is a whole number or zero.
            if myNum_coefficientPart.contains("E") {
                myNum_coefficientPart.removeLast(2)
            }
            
            let exponentEIndex      = myNum_sciString.firstIndex(of: "E") // will need to access original 'myNum_sciString' with it's "E" part.
            var myNum_exponent      = myNum_sciString.suffix(from: exponentEIndex!)
            _                       = myNum_exponent.removeFirst()      // removes, and returns, first element
            let exponentWithoutE    = String(myNum_exponent)
            
            print("\(myNum_coefficientPart) x 10E\(exponentWithoutE)")
//            0 x 10E0
//            1 x 10E0
//            1.2 x 10E0
//            1.2 x 10E0
//            1.2 x 10E0
//            1.2 x 10E3
//            1.2 x 10E3
//            1.2 x 10E-1
//            1.2 x 10E-2
//            1.2 x 10E-3
            
            // Then
            XCTAssert(myNum_beforeDecimal.count != 0, "There's nothing to represent the decimal part!")
            XCTAssert(exponentWithoutE != "", "There's no exponent value")
        }
    }
```
<br>
</br>


### Sample code structure for App Navigation:

```swift
// MARK: - FBNavigationDelegate Methods
extension FBMainSplitVC: FBNavigationDelegate {
    
    /// Set the middleVC based on selected Navigation Bar in leftVC.
    func setMiddleVC_forNavBar(at navBarIndex: kNavBarIndex) {
        var middleVCs: (graph: FBMiddleGraphVC, list: FBMiddleListVC)
        switch navBarIndex {
            case .library:      middleVCs = libraryGraphAndFoodListVCs()
            case .favorites:    middleVCs = favoritesGraphAndFoodListVCs()
            case .diary:        middleVCs = diaryGraphAndFoodListVCs()
            case .timeChart:    middleVCs = timeChartGraphAndFoodListVCs()
            case .trash:        middleVCs = trashGraphAndFoodListVCs()
        }
        setMiddleVCSplitViewItems(topVC: middleVCs.graph, bottomVC: middleVCs.list)
    }
    
    
    /// Sets up Library Graph and Library FoodList view controllers.
    ///
    /// A "mini" library of all_SRRFoods_library is loaded from disk.
    func libraryGraphAndFoodListVCs() -> (FBMiddleGraphVC, FBMiddleListVC) {
        let graphVC             = FBLibraryGraphVC(chartView: FBPieGraphView(frame: .zero))
        let graphTopToolBarVC   = FBGraphTopToolBarVC(identifier: .topToolBarVCID_libraryGraph)
        
        let listVC              = FBLibraryListVC(fdrfoods: all_FDRFoods_library,
                                                  identifier: .listVCID_library)
        let listTopToolBarVC    = FBLibraryListTopToolBarVC()
        let listBottomToolBarVC = FBFoodListBottomToolBarVC(popUpTitles: kFoodCategoryName.allNames)
        
        setGraphVC_FoodListVC_infoPanel_Delegates(graphVC: graphVC, 
                                                  graphTopToolBarVC: graphTopToolBarVC,
                                                  listVC: listVC,
                                                  listTopToolBarVC: listTopToolBarVC,
                                                  listBottomToolBarVC: listBottomToolBarVC)
        
        let topVC               = FBMiddleGraphVC(graphVC: graphVC, 
                                                  topToolBarVC: graphTopToolBarVC,
                                                  foodListTopToolBarVC: listTopToolBarVC)
        let bottomVC            = FBMiddleListVC(listVC: listVC,
                                                 topToolBarVC: listTopToolBarVC,
                                                 bottomToolBarVC: listBottomToolBarVC)
        return (topVC, bottomVC)
    }
    ... ...
```
<br>
</br>

# Sample Screen Shots
<table>
<tr>
	<td>
	<img src="images/screenshots/01a iFTmacOS main window Diary light.png" alt="blue marble weather daily forecast" width="500"/>
	</td>
	<td>
	<img src="images/screenshots/01b iFTmacOS main window Diary dark.png" alt="blue marble weather daily forecast" width="500"/>
	</td>
</tr>
<tr>
	<td>
	<img src="images/screenshots/02a iFTmacOS main window TimeChart light.png" alt="blue marble weather daily forecast" width="500"/>
	</td>
	<td>
	<img src="images/screenshots/05a iFTmacOS main window Trash light.png" alt="blue marble weather daily forecast" width="500"/>
	</td>
</tr>
<tr>
	<td>
	<img src="images/screenshots/08a iFTmacOS Settings Appearance light.png" alt="blue marble weather daily forecast" width="500"/>
	</td>
	<td>
	<img src="images/screenshots/06a iFTmacOS Settings Advanced light.png" alt="blue marble weather daily forecast" width="500"/>
	</td>
</tr>
<tr>
	<td>
	<img src="images/screenshots/08a iFTmacOS Settings Appearance light.png" alt="blue marble weather daily forecast" width="500"/>
	</td>
	<td>
	<img src="images/screenshots/012a Settings Window Demo light.png" alt="blue marble weather daily forecast" width="400"/>
	</td>
</tr>
</table>

<!-- ![readme-bluemarbleweather-daily](../images/BMW%20MainView%20Daily.png) -->

### This App required many long nights of implementations that would not have survived without the passion to keep it going. I've done the following:
* Model-View-Controller Design Pattern
* Delegates and Protocols, NSViewController/NSView
* Navigation Split Views
* Unit Testing
* Code documentation, DocC
* Error handling
* Toolbar and Menu selections
* Settings Window with TabBar subviews
* Help Book manual
* Alerts
* Empty states
* Random Generation of Sample Test Data
* Username/Password validation
* Text input validation
* Generics for Core Data objects
* Light and Dark Mode Selections
* Privacy Manifest
* Project organization


# Future Considerations
* Sync with HealthKit, for newer macOS versions
* Implement Core Data's CloudKit syncing
* InterOp with SwiftUI, to implement SwiftCharts for newer macOS versions


