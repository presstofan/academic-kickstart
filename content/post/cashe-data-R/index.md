---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Cache Time-consuming Computations in R"
subtitle: ""
summary: "Quick demonstration of caching time-consuming computations in R"
authors: [databentobox]
tags: [R, caching]
categories: [R]
date: 2016-08-07T17:30:42Z
lastmod: 2016-08-07T17:30:42Z
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

## Introduction

An R function that is able to cache potentially time-consuming computations could be useful in many cases. For example, taking the mean of a numeric vector is typically a fast operation. However, for a very long vector, it may take too long to compute the mean, especially if it has to be computed repeatedly (e.g. in a loop). If the contents of a vector are not changing, it may make sense to cache the value of the mean so that when we need it again, it can be looked up in the cache rather than recomputed.

## Example

Matrix inversion is computationally costly. In this example, we will try to cache an inverse matrix. The `<<-` operator which can be used to assign a value to an object in an environment that is different from the current environment. Below are two functions that are used to create a special object that stores a matrix and caches its inverse matrix.

The first function, `makeCacheMatrix` creates a special "matrix", which is really a list containing a function to

1. set the value of the matrix
2. get the value of the matrix
3. set the value of the inverse matrix
4. get the value of the inverse matrix

```r
makeCacheMatrix <- function(x = matrix()) {
  i <- NULL
  set <- function(y) {
    x <<- y
    i <<- NULL
  }
  get <- function() x
  setinverse <- function(inverse) i <<- inverse
  getinverse <- function() i
  list(set = set,
       get = get,
       setinverse = setinverse,
       getinverse = getinverse)
}
```

This function will take the object (list of functions) returned by `makeCacheMatrix()`. First, it call `getinverse()`. If there is an inverse matrix stored in the object, i.e. `i` is not `NULL`, then it will return `i` as the result. If no inverse matrix is stored in the object, it will call `get()` and get the matrix stored in the object. Then an inverse matrix is calculated and stored back into the object by calling `setinverse()` and then return the inverse matrix.

```r
cacheSolve <- function(x, ...) {
  ## Return a matrix that is the inverse of 'x'
  i <- x$getinverse()
  if(!is.null(i)) {
    message("cached inverse matrix found, getting the matrix...")
    return(i)
  }
  data <- x$get()
  i <- solve(data, ...)
  x$setinverse(i)
  i
}
```

Here is an example of how to use the codes above:

```r
# Testing
special_matrix <- makeCacheMatrix(matrix(1:4, 2, 2))
cacheSolve(special_matrix)
```