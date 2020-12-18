# Project-6-Code-Optimization Report

## Rotation Algorithm

Some Machine-Independent Optimizations Techniques that I have tried to implement and check the efficency for are:

- Code Motion
- Loop unrolling
- Blocking

The code we are provided with:

```c
for (j = 0; j < dim; j++)
	for (i = 0; i < dim; i++)
		dst[RIDX(dim-1-j, i, dim)] = src[RIDX(i, j, dim)];
```

Here is the detailed analysis for each of the technique:

### Code Motion

```c
/** Code motion*/
int k;
for (j = 0; j < dim; j++){
	k = dim - 1 -j;
	for (i = 0; i < dim; i++){
		dst[RIDX(k, i, dim)] = src[RIDX(i, j, dim)];
	}
}
```

In this piece of optimization, I have moved `dim - 1 - j` outside of the inner loop. As we can see that it is independent to the value of `i` iterator so there is no need to calculate `dim*dim` number of times, instead this optimizes code to calculate it `dim` number of times. This preserves the correctness of the program as the calculation of same thing is only `j` dependent so it is shifted in the outer loop.

The Average number of cycles we can reduce down to, can be seen in the following output:

```c
Testing Rotate:
          Time in milliseconds      Cycles used
==========================================================
Dimension naive_rotate my_rotate    naive_rotate my_rotate
==========================================================
512       1730         1414         4143814      3387910
1024      7643         6457         18306204     15467098
2048      87979        80807        210668693    193495898
4096      356465       342408       853555120    819889873
```

Upon taking the average of the used Cycles for these 4 image sizes, we can see it improves the code efficiency by only `5.01%`. This is not the efficiency we expect, so we will need to implement another method. It did not run faster because the number of times memory is accessed from memory instead of cache remains the same. Lower clock cycles would require low cache misses and that can be obtained by accessing data from cache rather than the memory.

### Loop unrolling

```c
/** 8 loop unrolling*/
for(i = 0; i < dim; i++){
	for(j = 0; j < dim; j += 8) {
		dst[RIDX(dim-1-j,i,dim)] = src[RIDX(i,j,dim)];
		dst[RIDX(dim-1-j-1,i,dim)] = src[RIDX(i,j+1,dim)];
		dst[RIDX(dim-1-j-2,i,dim)] = src[RIDX(i,j+2,dim)];
		dst[RIDX(dim-1-j-3,i,dim)] = src[RIDX(i,j+3,dim)];
		dst[RIDX(dim-1-j-4,i,dim)] = src[RIDX(i,j+4,dim)];
		dst[RIDX(dim-1-j-5,i,dim)] = src[RIDX(i,j+5,dim)];
		dst[RIDX(dim-1-j-6,i,dim)] = src[RIDX(i,j+6,dim)];
		dst[RIDX(dim-1-j-7,i,dim)] = src[RIDX(i,j+7,dim)];
	}
}
```

In this optimization technique I have unrolled the loop to perform these Load and Store entries in one go, 8 times every loop. This keeps the data more accessible by the compiler and reduces the time cost for the maintenance of loop increment. The number of times the program checks the value for `i` and incrementing it is reduced, in sum reducing the loop overhead. This preserves the correctness of the program since it is only unrolling what the loop would be executing. Rather than running loop 8 times each for same instruction, it is expanded out and written in lines of order. The loop is then incremented by spaces of 8 each time.

The Average number of cycles we can reduce down to, can be seen in the following output:

```c
Testing Rotate:
          Time in milliseconds      Cycles used
==========================================================
Dimension naive_rotate my_rotate    naive_rotate my_rotate
==========================================================
512       1627         1298         3899877      3111736
1024      8245         6242         19771887     14951904
2048      109317       60095        261797338    143917844
4096      390126       277469       936646822    664473884
```

Upon taking the average of the used Cycles for these 4 image sizes, we can see it improves the code efficiency by only `16.2%`. The code efficiency is still not up to the mark. Since the dimension for the image is a multiple of 32, we can now try loop unrolling of size 16 and 32 alternatively.

```c
/** 16 loop unrolling*/
for(i = 0; i < dim; i++){
	for(j = 0; j < dim; j += 16) {
		dst[RIDX(dim-1-j,i,dim)] = src[RIDX(i,j,dim)];
		dst[RIDX(dim-1-j-1,i,dim)] = src[RIDX(i,j+1,dim)];
		dst[RIDX(dim-1-j-2,i,dim)] = src[RIDX(i,j+2,dim)];
		dst[RIDX(dim-1-j-3,i,dim)] = src[RIDX(i,j+3,dim)];
		dst[RIDX(dim-1-j-4,i,dim)] = src[RIDX(i,j+4,dim)];
		dst[RIDX(dim-1-j-5,i,dim)] = src[RIDX(i,j+5,dim)];
		dst[RIDX(dim-1-j-6,i,dim)] = src[RIDX(i,j+6,dim)];
		dst[RIDX(dim-1-j-7,i,dim)] = src[RIDX(i,j+7,dim)];
		dst[RIDX(dim-1-j-8,i,dim)] = src[RIDX(i,j+8,dim)];
		dst[RIDX(dim-1-j-9,i,dim)] = src[RIDX(i,j+9,dim)];
		dst[RIDX(dim-1-j-10,i,dim)] = src[RIDX(i,j+10,dim)];
		dst[RIDX(dim-1-j-11,i,dim)] = src[RIDX(i,j+11,dim)];
		dst[RIDX(dim-1-j-12,i,dim)] = src[RIDX(i,j+12,dim)];
		dst[RIDX(dim-1-j-13,i,dim)] = src[RIDX(i,j+13,dim)];
		dst[RIDX(dim-1-j-14,i,dim)] = src[RIDX(i,j+14,dim)];
		dst[RIDX(dim-1-j-15,i,dim)] = src[RIDX(i,j+15,dim)];
	}
}
```

Checking the results we get:

```c
Testing Rotate:
          Time in milliseconds      Cycles used
==========================================================
Dimension naive_rotate my_rotate    naive_rotate my_rotate
==========================================================
512       1566         1414         3751693      3390438
1024      10833        6556         25945048     15703080
2048      89467        60732        214242675    145429483
4096      374226       281265       896077520    673485349
```

Taking the averages of the 4 clock cycles, we see that the code has been optimized by `26.5%`. This is indeed an effective way of optimizing the code time efficiency.  However, Pipelined processors and loop unrolling are dependent on the general architecture of machine CPU. So we must look for optimizations that are compiler optimizations and machine independent. This type of loop unrolling has a disadvantage of higher Instruction Count.

### Blocking

It is the optimization technique learnt in lecture. Going down all columns and rows sequentially can be very cost inefficient. This method will improve the overall temporal locality by accessing `blocks` of data repeatedly.

The optimization technique that worked by far the best and most efficient was implementing 16x blocking through the loop, which results in cache perfomance.

```c
unsigned int a, b;
for(a=0; a < dim; a+=16){
	for(b=0; b < dim; b+=16){
		for(i=a; i < a+16; i++) {
			for(j=b; j < b+16; j++) {
				dst[RIDX(dim-1-j,i,dim)] = src[RIDX(i,j,dim)];
			}
		}
	}
}
```

This algorithm breaks down the array into blocks of 16 so it can fill the cache block. Keeping the Row-major order for the source array will implement spatial locality which reduces the number of miss. So when each time the destination array is accessed in Column-major order, the source array is still in cache. This will lead us to having as few as almost `16` cache misses each time.

I have implemented this method because this takes memory accessing time and clock cycles into consideration. Since we are dealing with lots of load and store of explicitly larger sized arrays in nested loops, it is most efficient to adjust the array loops in cache memory than the direct memory itself. This will ultimately reduce the number of cache misses, and use lesser cycles, increasing the overall performance of the code.

When compiled we get the following results:

```c
Testing Rotate:
          Time in milliseconds      Cycles used
==========================================================
Dimension naive_rotate my_rotate    naive_rotate my_rotate
==========================================================
512       1357         1224         3257976      2934029
1024      7072         5564         16939007     13327958
2048      80585        45221        192966796    108287111
4096      338443       184585       810399700    441987490
```

After analyzing the result we can see a significant reduction in number of clock cycles used to execute the program. This shows that indeed improvement has been made in terms of cache hits and misses. Averages show us that our program is about `44.6%` more efficient!

## Smooth Algorithm

The code we are provided with:

```c
for (j = 0; j < dim; j++)
	for (i = 0; i < dim; i++)
		dst[RIDX(i, j, dim)] = avg(dim, i, j, src);
```

Here is the detailed analysis for each of the technique I have used

### Loop Interchange

```c
for (i = 0; i < dim; i++){
	for (j = 0; j < dim; j++){
		dst[RIDX(i, j, dim)] = avg(dim, i, j, src);
	}
}
```

This type of optimization will change the nesting of loops to access data in order stored in memory. This can be achieved by simply interchanging the nexted `for loop` statements.

The results we get after implementing Loop Interchange technique are:

```c
Testing Smooth:
          Time in milliseconds      Cycles used
==========================================================
Dimension naive_smooth my_smooth    naive_smooth my_smooth
==========================================================
256       18830        18076        45095488     43292233
512       81455        75527        195049221    180855943
1024      418556       312824       1002242567   749056168
2048      2538873      1559780      6079258730   3734843340
```

We can see that there is a change in number of clock cycles used when the loop statments are interchanged. The nested loop in naive_smooth function was accessing memory in a non sequential order, simpley interchanging the loop statements will result in accessing the data in a sequential order. This preserves the correctness of the program since, now it is sequential access instead of striding through memory every `dim` times. This improves the spatial locality of the program.

We can analyze an improvement of code efficiency by `35.6%`. This reaches the 30% code efficency improvement we are looking for, but we still have room for improvement. We see there are multiple finctions being called inside eachother. To fix that we will implement another technique building up on loop interchange optimization that we just wrote.

### Function Inlining

One function call in `c` is equivalent to a number of instructions in machine code. Each time a function is called, there are `book keeping` records added onto the stack. This includes the Activation records and allocates more memory. This would use more registers as well since it returns back to the memory address where the function was called. In machine code after every function call, the stack is popped and pushed so adding to the `IC`.

```c
/** Loop interchange*/
for (i = 0; i < dim; i++){
	for (j = 0; j < dim; j++){
		/** avg()*/
		int ii, jj;
		pixel_sum sum;
		pixel current_pixel;

		initialize_pixel_sum(&sum);
		for(ii = maximum(i-1, 0); ii <= minimum(i+1, dim-1); ii++){
			for(jj = maximum(j-1, 0); jj <= minimum(j+1, dim-1); jj++) {
				/** accumulate_sum()*/
				sum.red += (int) src[RIDX(ii, jj, dim)].red;
				sum.green += (int) src[RIDX(ii, jj, dim)].green;
				sum.blue += (int) src[RIDX(ii, jj, dim)].blue;
				sum.num++;
			}
		}
		/** assign_sum_to_pixel()*/
		current_pixel.red = (unsigned short) (sum.red/sum.num);
		current_pixel.green = (unsigned short) (sum.green/sum.num);
		current_pixel.blue = (unsigned short) (sum.blue/sum.num);

		dst[RIDX(i, j, dim)] = current_pixel;
	}
}
```

In this code I have implemented `Function Inlining` so that no function is called inside the loop. The instructions of function `avg()` are added in line. I have removed the return values. Then the two functions `accumulate_sum()` and `assign_sum_to_pixel()` are also written in line to avoid any function call. I have changed the `->` pointer to `.` pointer because the value was passed by reference in the parameter. 

By using `Function Inlining` technique for optimization, I achieved and overall improvement in perfomance. It removed all the bookkeeping instructions, instructions to Push arguments to stack, instructions to Push frame pointer, instructions to Pop return value, reset frame pointer and Pop return address. All of these instructions times `dim*dim` instruction count have been reduced improving the code efficiency of this smooth function.

The results we get are:

```c
Testing Smooth:
          Time in milliseconds      Cycles used
==========================================================
Dimension naive_smooth my_smooth    naive_smooth my_smooth
==========================================================
256       12479        10011        29886915     23976092
512       77332        39869        185175219    95471239
1024      253839       166803       607813167    399409712
2048      1219843      657657       2920872424   1574743394
```

We can observe an overall `44%` of improvement in our efficiency, which is a lot better. So in conclusion `Sequential Memory Access` in combination with `Function Inlining` has improved the code efficiency and this is the algorithm I have implemented in `my_smooth()` function.
