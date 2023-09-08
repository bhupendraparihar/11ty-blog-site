---
title: Intersection Observer API
author: Bhupendra Parihar
date: 2023-09-08 16:15:32
tags: ['post', 'featured']
image: /assets/blog/code1.jpg
description: Develop a lazy loading is very easy with Intersection Observer API. In this article we will learn how to use it with lazy loading of images in a e-commerce web application.
---

Before the Intersection Observer API became widely supported, lazy loading of images and other content could be achieved using alternative techniques. One common approach was to use the `window.onscroll` event handler to determine when an element was in or near the viewport. Here's a simplified example of how you could implement lazy loading before the Intersection Observer API:

```javascript
import React, { useState, useEffect } from 'react';

function LazyLoadImage({ src, alt }) {
  const [isVisible, setIsVisible] = useState(false);

  useEffect(() => {
    const handleScroll = () => {
      const element = document.querySelector('.lazy-load-image');
      if (element) {
        const elementTop = element.getBoundingClientRect().top;
        const viewportHeight =
          window.innerHeight || document.documentElement.clientHeight;
        if (elementTop <= viewportHeight) {
          setIsVisible(true);
        }
      }
    };

    // Attach the scroll event listener
    window.addEventListener('scroll', handleScroll);

    // Check initial visibility
    handleScroll();

    // Cleanup by removing the event listener
    return () => {
      window.removeEventListener('scroll', handleScroll);
    };
  }, []);

  return (
    <img src={isVisible ? src : ''} alt={alt} className="lazy-load-image" />
  );
}

export default LazyLoadImage;
```

Let's see, how we can write a product component in a typical e-commerce application

```javascript
import { Link } from 'react-router-dom';
import Card from 'react-bootstrap/Card';
import Button from 'react-bootstrap/Button';
import Rating from '../Rating/Rating';
import axios from 'axios';
import { useContext } from 'react';
import { Store } from '../../Store';

function Product(props) {
  const { product } = props;
  const { state, dispatch: ctxDispatch } = useContext(Store);
  const {
    cart: { cartItems },
  } = state;

  const addToCartHandler = async (item) => {
    const existItem = cartItems.find((x) => x._id === product._id);
    const quantity = existItem ? existItem.quantity + 1 : 1;
    const { data } = await axios.get(`/api/products/${item._id}`);

    if (data.countInStock < quantity) {
      window.alert('Sorry, Product is out of stock');
      return;
    }
    ctxDispatch({
      type: 'CART_ADD_ITEM',
      payload: { ...item, quantity },
    });
  };

  return (
    <Card className="product" key={product.slug}>
      <Link to={`/product/${product.slug}`}>
        <img src={product.image} className="card-img-top" alt={product.name} />
      </Link>
      <Card.Body>
        <Link to={`/product/${product.slug}`}>
          <Card.Title>{product.name}</Card.Title>
        </Link>
        <Rating rating={product.rating} numReviews={product.numReviews} />
        <Card.Text>${product.price}</Card.Text>
        {product.countInStock === 0 ? (
          <Button variant="light" disabled>
            Out of Stock
          </Button>
        ) : (
          <Button onClick={() => addToCartHandler(product)}>Add to Cart</Button>
        )}
      </Card.Body>
    </Card>
  );
}

export default Product;
```

The above code create a Product component and consume the props which holds a product object.

The problem statement for lazy loading is that suppose we have 100s of product and all these product will try to load the image even they are not there in the view port. So we need to write a logic such that only load the image if the section is in view port or near to the view port.

So let's create two variable, isIntersecting and imageURL

```javascript
const [isIntersecting, setIsIntersecting] = useState(false);
const [imageUrl, setImageUrl] = useState('');
```

and image element src should point to imageURL instead of product.image

```javascript
<img src={imageUrl} className="card-img-top" alt={product.name} ref={ref} />
```

Now, if isIntersecting is true, lets update the imageUrl with product.image

```javascript
useEffect(() => {
  if (isIntersecting) {
    setImageUrl(product.image);
  }
}, [isIntersecting, imageUrl, product.image]);
```

But how do we find the right value of isIntersecting, the answer is Intersection Observer API which came into picture in 2016.

```javascript
useEffect(() => {
  const observer = new IntersectionObserver(([entry]) => {
    setIsIntersecting(entry.isIntersecting);
  });
  observer.observe(ref.current);
  return () => observer.disconnect();
}, [isIntersecting]);
```

The combined code will looks like below code.

```javascript
import { Link } from 'react-router-dom';
import Card from 'react-bootstrap/Card';
import Button from 'react-bootstrap/Button';
import Rating from '../Rating/Rating';
import axios from 'axios';
import { useContext, useEffect, useRef, useState } from 'react';
import { Store } from '../../Store';

function Product(props) {
  const { product } = props;
  const { state, dispatch: ctxDispatch } = useContext(Store);
  const {
    cart: { cartItems },
  } = state;

  const [isIntersecting, setIsIntersecting] = useState(false);
  const [imageUrl, setImageUrl] = useState('');

  const ref = useRef(null);

  useEffect(() => {
    const observer = new IntersectionObserver(([entry]) => {
      setIsIntersecting(entry.isIntersecting);
    });
    observer.observe(ref.current);
    return () => observer.disconnect();
  }, [isIntersecting]);

  useEffect(() => {
    if (isIntersecting) {
      setImageUrl(product.image);
    }
  }, [isIntersecting, imageUrl, product.image]);

  const addToCartHandler = async (item) => {
    const existItem = cartItems.find((x) => x._id === product._id);
    const quantity = existItem ? existItem.quantity + 1 : 1;
    const { data } = await axios.get(`/api/products/${item._id}`);

    if (data.countInStock < quantity) {
      window.alert('Sorry, Product is out of stock');
      return;
    }
    ctxDispatch({
      type: 'CART_ADD_ITEM',
      payload: { ...item, quantity },
    });
  };

  return (
    <Card className="product" key={product.slug}>
      <Link to={`/product/${product.slug}`}>
        <img
          src={imageUrl}
          className="card-img-top"
          alt={product.name}
          ref={ref}
        />
      </Link>
      <Card.Body>
        <Link to={`/product/${product.slug}`}>
          <Card.Title>{product.name}</Card.Title>
        </Link>
        <Rating rating={product.rating} numReviews={product.numReviews} />
        <Card.Text>${product.price}</Card.Text>
        {product.countInStock === 0 ? (
          <Button variant="light" disabled>
            Out of Stock
          </Button>
        ) : (
          <Button onClick={() => addToCartHandler(product)}>Add to Cart</Button>
        )}
      </Card.Body>
    </Card>
  );
}

export default Product;
```

But this code also looks like our onScroll function, so what is the advantage.

Here's why the Intersection Observer API is better than the `window.onscroll` approach:

1. Efficiency:

   - The Intersection Observer API is designed to be highly efficient. It allows you to observe multiple elements simultaneously without causing a significant performance hit because it utilizes browser internals to handle intersection checks.
   - In contrast, the `window.onscroll` approach often involves repeatedly calculating element positions and firing scroll events, which can lead to performance bottlenecks, especially when observing many elements or complex scrolling behavior.

2. Simplicity:

   - With the Intersection Observer API, you simply define a callback function to be executed when an element intersects with the viewport or a specified container. This makes the code cleaner and more maintainable.
   - The `window.onscroll` approach requires you to manage a lot of manual calculations and event handling, making the code less straightforward.

3. Flexibility:

   - The Intersection Observer API provides options for configuring the observed elements, such as specifying a margin around the intersection area, toggling once or multiple times, and observing elements inside specific containers.
   - With `window.onscroll`, achieving similar functionality often requires complex custom code and calculations.

4. Better Performance:

   - Intersection observers are optimized for performance and leverage browser-native code for intersection calculations, resulting in smoother scrolling and better overall user experience.
   - The `window.onscroll` approach can lead to janky scrolling and increased CPU usage, especially when dealing with a large number of elements or complex logic.

5. Reducing JavaScript Execution:
   - Since the Intersection Observer API relies on callbacks, it allows for better separation of concerns. JavaScript is only executed when intersections occur, reducing unnecessary script execution.
   - The `window.onscroll` approach may involve continuous script execution as the user scrolls, even when no intersections are happening, potentially wasting resources.

I hope you enjoyed the article.
