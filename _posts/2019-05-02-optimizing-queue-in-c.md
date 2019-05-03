---
layout: post
title: Optimizing queue data structure in C
---

## WIP

The way a queue works is the data enters first will be the first to leave.

## Results

Add to front:
total_element_size: 40000, total_pointer_size: 80000 total_size: 120000 total_time: 0.000612

Add to end (naive):
total_element_size: 40000, total_pointer_size: 80000 total_size: 120000 total_time: 0.160356

Add to end (using a end tracker):
total_element_size: 40000, total_pointer_size: 80000 total_size: 120000 total_time: 0.000610



### Code
```cpp
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

struct node_t {
  int val;
  struct node_t *next;
};

struct node_t *node_list;
struct node_t *last_node;

static unsigned long total_size;
static unsigned long total_pointer_size;
static unsigned long total_element_size;

void *allocate_node() {
  total_pointer_size += sizeof(struct node_t *);
  total_element_size += sizeof(int);
  total_size = total_pointer_size + total_element_size;
  return malloc(sizeof(struct node_t));
}

void add_node_to_front(int val) {
  struct node_t *new_node = (struct node_t *)allocate_node();
  new_node->next = node_list;
  new_node->val = val;
  node_list = new_node;
}

void add_node_to_end(int val) {
  struct node_t *new_node = (struct node_t *)allocate_node();
  new_node->val = val;
  new_node->next = NULL;
  if (node_list == NULL) {
    node_list = new_node;
    last_node = node_list;
    return;
  }
  last_node->next = new_node;
  last_node = new_node;
}

void add_node_to_end_naive(int val) {
  struct node_t *new_node = (struct node_t *)allocate_node();
  new_node->val = val;
  new_node->next = NULL;
  if (node_list == NULL) {
    node_list = new_node;
    return;
  }
  struct node_t *tmp_node = node_list;
  while (tmp_node->next) {
    tmp_node = tmp_node->next;
  }
  tmp_node->next = new_node;
}

int main() {
  node_list = NULL;
  last_node = NULL;

  clock_t start, end;

  last_node = node_list;

  start = clock();

  for (int i = 0; i < 10000; i++) {
    //        add_node_to_end_naive(i);
    add_node_to_end(i);
    //        add_node_to_front(i);
  }

  end = clock();

  double total_time = (double)(end - start) / CLOCKS_PER_SEC;

  printf("total_element_size: %lu, total_pointer_size: %lu total_size: %lu "
         "total_time: %lf\n",
         total_element_size, total_pointer_size, total_size, total_time);
}
```
