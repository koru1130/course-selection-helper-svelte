<script lang="ts">
import GridCell from "./GridCell.svelte"
import { candidateCourses, selectedCourses } from "../../stores";
import type { SerNo } from "../../types";
import { getCourse } from "../../data/getCourse";
import { array } from 'fp-ts'
import { pipe } from 'fp-ts/lib/function'
import { cellWidth } from "../../consts";
import { debug } from "svelte/internal";
import { getMaxArraySize } from "../../utils";
const rows = ['一', '二', '三', '四', '五', '六']
const columns = ['0','1','2','3','4','5','6','7','8','9','10','A','B','C','D']
const 節次to時間 = {
    "0" :	"07:10 - 08:00",
    "1" :	"08:10 - 09:00",
    "2" :	"09:10 - 10:00",
    "3" :	"10:20 - 11:10",
    "4" :	"11:20 - 12:10",
    "5" :	"12:20 - 13:10",
    "6" :	"13:20 - 14:10",
    "7" :	"14:20 - 15:10",
    "8" :	"15:30 - 16:20",
    "9" :	"16:30 - 17:20",
    "10":	"17:30 - 18:20",
    "A" :	"18:25 - 19:15",
    "B" :	"19:20 - 20:10",
    "C" :	"20:15 - 21:05",
    "D" :	"21:10 - 22:00"
}

$: console.log("owo",$selectedCourses)



$: sortedCourses = (()=>{
    let arr
    // SerNo, TimeSection index
    : ([SerNo, number] | "PlaceHolder" )[][][]
    = [...Array(rows.length)].map( _ => [...Array(columns.length)].map( _ => [] ) ) 
    
    for (const serNo of $selectedCourses) {                    
        const course = getCourse(serNo)
        if(!course) continue;
        if(course.time == "N/A") continue;
        for (const [sectionId, timeSection] of course.time.entries()){                                
            const level =
            pipe(
                Array.from( Array(timeSection.duration).keys()),
                array.map( x => arr[timeSection.dayOfWeek][timeSection.startTime + x].length ), 
                array.reduce(0 ,Math.max)
            )
                
            arr[timeSection.dayOfWeek][timeSection.startTime][level] = [serNo, sectionId]
    
            for (let i = 0; i < timeSection.duration; i++) { 
                for (let j = 0; j < level+1; j++) {
                    if(!arr[timeSection.dayOfWeek][timeSection.startTime + i][j]){
                        arr[timeSection.dayOfWeek][timeSection.startTime + i][j] = "PlaceHolder"
                    }
                }
            }
        }
        
    }    
    console.log("sortedCourse updated", arr)
    return arr    
    
})()

$: _rows = rows.map( (x, i) => ({
    name: x, 
    h: Math.max(200, getMaxArraySize(sortedCourses[i])*40+5)
}))

</script>


<style>
    table {
        overflow: auto; 
        height: auto;
        white-space: nowrap;
        border-collapse: collapse;
        table-layout: fixed;
        display: block;
    }
    tbody > tr {
        height: 10em;      
        border-bottom: 1px solid;  
        overflow-y: scroll;
    }    
    
    td {
        min-width: 8em;
        border-right: 1px solid;
    }

    tr > th:not(thead > tr > :first-child){
        background-color: lightgray;
    }

    th {        
        text-align: center;
    }

    thead tr th {
        position: -webkit-sticky; /* for Safari */
        position: sticky;
        top: 0;
        background-color: lightgray;
        z-index: 2;        
    }
    
    tbody th {
        position: -webkit-sticky; /* for Safari */
        position: sticky;
        left: 0;
        background-color: lightgray;
        max-width: 1em;
        z-index: 2;        
    }

    thead th:first-child {
        left: 0;
        z-index: 3;
        background-color: white;
        min-width: 2em; 
    }


</style>

<table>    
    <thead>
        <tr>
            <th>
                
            </th>
            {#each columns as columnName}
                <th>                                        
                    {columnName}
                    <br>
                    <small>  {節次to時間[columnName]} </small>
                </th>
            {/each}
        </tr>
    </thead>
    <tbody>
    {#each _rows as {name, h}, i}
        <tr style="width: {columns.length * cellWidth} em; height: {h}px">
            <th style="height: {h}px">
                {name}
            </th>
            {#each columns as _, j}
                <td style="height: {h}px">                                    
                    <GridCell v={sortedCourses[i][j]} height={h} />
                </td>
            {/each}
        </tr>
    {/each}
    </tbody>
    
</table>