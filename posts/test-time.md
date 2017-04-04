# Test Time

A question I get asked pretty frequently is, 

> How much time do you typically estimate for writing unit tests?

Usually what people<sup>[1](#people)</sup> are looking for is something like, "I need X hours for production code, and Y hours for writing tests for this feature."  I think this is the wrong way to look at it, but I've struggled with explaining why.  I came a across a real example today that I think might be instructive.

We found a bug in some of our code that was using a list component; when removing an item from the list, the `value` argument passed to the event handler was null, but we were looking for `value.id`.  In addition, the POST body for our API call expected the properties to be defined even if they weren't set, and to have a default value of `-1`.

The total time to discover, log, reproduce and find the root cause of this bug was on the order of a few **hours**.

The **production** code to fix the bug looked like this:

```javascript
static getCategoryValue(val) {
    let category = -1;
    if (val && val.id !== undefined && val.id !== null) {
      category = val.id;
    }
    return category;
}
```

The total time to write the function was on the order of a few **minutes**.

The **test** code to check the behavior looked like this:

```javascript
  describe('Category values', () => {
    it('Returns -1 if the value is null', () => {
      expect(PoiForm.getCategoryValue(null)).to.equal(-1);
    });

    it('Returns -1 if the value is undefined', () => {
      expect(PoiForm.getCategoryValue()).to.equal(-1);
    });

    it('Returns -1 if the id is null', () => {
      const val = {id: null};
      expect(PoiForm.getCategoryValue(val)).to.equal(-1);
    });

    it('Returns -1 if the id is undefined', () => {
      const val = {};
      expect(PoiForm.getCategoryValue(val)).to.equal(-1);
    });

    it('Returns the id if it is 0', () => {
      const val = {id: 0};
      expect(PoiForm.getCategoryValue(val)).to.equal(0);
    });

    it('Returns the id if it is 1', () => {
      const val = {id: 1};
      expect(PoiForm.getCategoryValue(val)).to.equal(1);
    });
  })
```

The total time to write the above tests was on the order of a few **seconds**.

Of course, this is a very simple example, but I think the point still holds--as a general rule, it doesn't make sense to compare tests and production methods by lines of code, nor really even estimate them separately.


<a id="people">1</a>: Project managers are people too!
