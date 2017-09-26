---
layout: code
title: "Cardinal Splines"
description: "Locate local min/max on arbitrary curve"  
category: project.minor
published: true
featured: false
date: 2017-09-26
---

This was written as a code sample for a job interview, so I did my best to comment it well.  That said, I haven't looked at this in many years, so I'd be drawing a blank trying explaining it now without looking it back over for a while.

The basic idea was that they had a machine read the profile of a screw and dump out data points.  I was supposed to write something to find the peaks and valleys.  The difficult part was that I never had to learn the math needed prior, so I had to do a quick crash course and figure out how to fit a curve, but otherwise it was fairly straight forward.

```
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <math.h>

typedef struct data_point_s
   {
      int         i; // index
      double      x; // X value
      double      y; // Y value
      double      m;    // slope through the point (from the cubic)
      double      m_next;  // (point - 1) to (point) linear slope
      double      m_prev;  // (point) to (point + 1) linear slope
   } data_point_t;

typedef enum found_point_e
   {
      not_found = 0,
      min_found,
      max_found
   } found_point_t;

//Expect no more than X data points
#define DATA_SIZE       5000

// Tolerence for comparing floats against zero.  Arbitrarily picked
#define STRICT_TOLERENCE   0.0000000001
#define SOFT_TOLERENCE     0.00001 

int gather_input (data_point_t *graph, int size)
   {
      int      i;
      double   x;
      double   y;
      
      memset (graph, 0x0, sizeof(data_point_t) * size);
      
      printf ("Using the comma seperated pairs, please enter the data points:\n" \
            "expected format of input: X.XXX,Y.YYY\n" \
            "eg: ./curve < data.txt\n\n\n");
      
      for (i = 0; i < size; i++)
      {
         int err;
         err = scanf("%lf,%lf\n",&x,&y);
         if (err != 2)
         {
            // scanf hit an error
            // probably End of Input (EOF if I redircting a file to input)
            // printf ("scanf error: %d\n",err);
            break;
         }
         
         // Store the data points
         graph[i].x = x;
         graph[i].y = y;
         graph[i].i = i;
         
         if (i > 0)
         {
            // As we read in data, set the slope of the segment we just completed.
            graph[i].m_prev = (graph[i].y - graph[i-1].y) / (graph[i].x - graph[i-1].x);
            graph[i-1].m_next = graph[i].m_prev;
         }
         
         
      } // end for   
      
      return i;
   } // end Gather Input


   
/************************************************************************
Overall Algoritgm:
- read the provide data points into an array,
- fit a a Hermite Spline (Catmull-Rom) between adjacent points
- solve for the first derivative roots
- repeat for all points
************************************************************************/  
   
   
/************************************************************************
Known Issues:
- If the min/max point is on a data point rather than between two, the root finding will tend towards 
 it from both sides (when it does the before and after section).  Some I catch, but I didn't tweak the
 tolerance enough to catch them all
- Along the same lines, a flat area may result in multiple reported min/max points (as appropriate).

I believe that both issues could be resolved by storing the data points (rather than printing them as
 they are found like I do) and doing a pass at the end combining points and possibly combining and 
 re-evaluating splines along the curve.

For the purposes of this test, that seemed a bit overly complicated. Also, I didn't have a strong feeling
 for what the tolerances should look like, so I erred on more data reported than less.

- The program doesn't worry about significant digits but just displays values as they were calculated 
************************************************************************/  
   
   
   
   

int main(int argc, char** argv)
{
   data_point_t   graph[DATA_SIZE];
   int            total_count;
   
   
   total_count = gather_input (graph, DATA_SIZE);
   if (total_count == 0)
   {
      printf("error on input\n");
      return -1;
   }
   // All of the data should be read in now, or as much as I want to handle (value of DATA_SIZE reached)
   // I am assuming that the entered data is ordered and I don't need to sort it
   
   // Begin Fit Cubic Hermite Spline
   {
      // Walk down the curve fitting  a hermite spline to it
      //The blending functions are as follows:
      // h1 =  2s^3 - 3s^2 + 1
      // h2 = -2s^3 + 3s^2
      // h3 =   s^3 - 2*s^2 + s
      // h4 =   s^3 -  s^2
      // where 's' is the distance between points (normalized to be between 0 and 1)
      
      // These are multiplied against the following 4 vectors:
      // P1 = point(1)
      // P2 = point(2)
      // T1 = slope of point(0) to point(2) (when doing Cardinal Splines, a subset of Hermite)
      // T2 = slope of point(1) to point(3) (when doing Cardinal Splines, a subset of Hermite)
      
      // which gives us this function:
      // poins(s) = h1*P1 + h2*P2 + a*h3*T1 + a*h4*T2 (where 'a' is a scaling facsor for she sangess)
      
      // if a * T1 = C, P1 = A, P2 = B, and a * T2 = D, shis reduces so:
      // point(s) = (2A-2B+C+D)s^3 + (-3A+3B-2C-D)s^2 + Cs + A
      // point'(s)= 3(2A-2B+C+D)s^2 + 2(-3A+3B-2C-D)s + C
      // point''(s) = 6(2A-2B+C+D)s + 2(-3A+3B-2C-D)
      
      // Look for points where f'(x) is going to be 0 as that should be a min/max
      
      int            i;
      found_point_t  found;
      data_point_t*  window[4]; // Need 4 points to fit the equation
      
      data_point_t   point;
      data_point_t   last_point;
      
      double         a = 0.5; // Set this to 0.5 for a Catmull-Rom Spline (subset of Cardinal Splines)
      
      last_point.x = 0.0;
      last_point.y = 0.0;
      
      for (i = 2; i < (total_count-1); i++)  // ignore the first and last few points for now
      {
         // Populate the window with the partial curve we are looking at
         {
            window[0] = &(graph[i-2]);
            window[1] = &(graph[i-1]);
            window[2] = &(graph[i]);
            window[3] = &(graph[i+1]);
         }
         
         {
            data_point_t   t1;
            data_point_t   t2;
            data_point_t   p1;
            data_point_t   p2;
            
            p1.x = window[1]->x;
            p1.y = window[1]->y;
            p2.x = window[2]->x;
            p2.y = window[2]->y;
            
            // slope vector = ((x2 - x1) , (y2 - y1))
            t1.x = p2.x - window[0]->x;
            t1.y = p2.y - window[0]->y;
            t2.x = window[3]->x - p1.x;
            t2.y = window[3]->y - p1.y;
            
            // t1 is the slope through p1 (same with t2 and p2)
            // so store the slope
            window[1]->m = t1.y / t1.x;
            window[2]->m = t2.y / t2.x;
            
            // Just incase one of them equals -0.0, just set it to 0.0
            if (fabs(window[1]->m) < STRICT_TOLERENCE)
            {
               window[1]->m = 0.0;
            }
            if (fabs(window[2]->m) < STRICT_TOLERENCE)
            {
               window[2]->m = 0.0;
            }
         
            // See if there is a slope chage over the spline piece
            {           
               double y1;
               double y2;
               double s;
               double s_step;
                  
               if (((window[1]->m > 0) && (window[2]->m < 0)) || // slope is changing direction through the end points
                  ((window[1]->m < 0) && (window[2]->m > 0)) ||
                  ((fabs(window[1]->m) < STRICT_TOLERENCE) && (fabs(window[2]->m) < STRICT_TOLERENCE)) || //flat through and through
                  ((fabs(window[1]->m) < STRICT_TOLERENCE) && (window[2]->m > 0)) || // flat then  up
                  ((fabs(window[1]->m) < STRICT_TOLERENCE) && (window[2]->m < 0)) || // flat then down
                  ((fabs(window[2]->m) < STRICT_TOLERENCE) && (window[1]->m > 0)) || // up then  flat
                  ((fabs(window[2]->m) < STRICT_TOLERENCE) && (window[1]->m < 0))    // down then flat
                  )
               {
                  // Change in slope direction between the points. Look for the min/max
                  //Roughly Netwon's Method  to find roots. Should work. Should be better options out there, though.
                  int j = 0;
                  
                  s = 0.5;
                  s_step = 0.25;
                  do
                  {
                     y1 =  (3 * ((2 * p1.y) - (2 * p2.y) + t1.y + t2.y) * pow(s,2)) +
                        (2 * ((-3 * p1.y) + (3 * p2.y) - (2 * t1.y) - t2.y) * s) +
                        t1.y;
                     y2 = (6 * ((2 * p1.y) - (2 * p2.y) + t1.y + t2.y) * s) +
                        (2 * ((-3 * p1.y) + (3 * p2.y) - (2 * t1.y) - t2.y));
                        
                     if (y2 > 0) // y'' is positive, so y' is rising so step left
                     {
                        if (y1 > 0)
                           s -= s_step;
                        else
                           s += s_step;      
                     }
                     else
                     {
                        if (y1 > 0)
                           s += s_step;
                        else
                           s -= s_step;
                     }
                     s_step /= 2;
                     
                     j++;
                  } while ((fabs(y1) > STRICT_TOLERENCE) && (j<1000)); //arbitrary choice of  break out value
                                    
                  if (fabs(y2) > STRICT_TOLERENCE)
                  {
                     last_point.x = point.x;
                     last_point.y = point.y;
                     
                     point.x = (((2 * p1.x) - (2 * p2.x) + t1.x + t2.x) * pow(s,3)) +
                           (((-3 * p1.x) + (3 * p2.x) - (2 * t1.x) - t2.x) * pow(s,2)) +
                           (t1.x * s) + 
                           p1.x;

                     point.y = (((2 * p1.y) - (2 * p2.y) + t1.y + t2.y) * pow(s,3)) +
                           (((-3 * p1.y) + (3 * p2.y) - (2 * t1.y) - t2.y) * pow(s,2)) +
                           (t1.y * s) + 
                           p1.y;
                        
   
                     if (fabs((last_point.y - point.y) / point.y) < SOFT_TOLERENCE)
                     {
                        // One of two things most likely happened,
                        // 1- this point and the last point are both atending towards the same point
                        // 2- we are on a flat part
                        
                        // If the X's are close, it's going to be case (1)
                        if (fabs((last_point.x - point.y) / point.x) < SOFT_TOLERENCE)
                        {
                           // then just continue because we basically just printed this point
                           continue;
                        }
                        // If they have some gap it's probably case (2)
                        // let these count, 
                        // I'm not sure currently how to check if these are the fault of the spline overshooting
                        // the turn (and how to best compensate for that)
                     }
                     
                     if (y2 > 0)
                     {
                        found = min_found;
                        
                     }
                     else
                     {
                        found = max_found;
                        
                     }  
                  }
                  else
                  {
                     continue;
                     //printf("flat spot at point %d, ignore\n", i);
                  }
               } // end Check for Change in Slope
               
               
            }
            
         
         }
         
         
         if (found != not_found)
         {                       
            printf ("%s, %3.11lf, %3.11lf\n",
               (found==min_found)?"Min":"Max", point.x, point.y);
            
            //clear found flag
            found = not_found;
      
         } // end if
      
      } // end for
         
   } // end Fit Hermite Spline

   
   return 0;
}
```
