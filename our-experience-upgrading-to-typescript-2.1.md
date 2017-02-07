# Our Experiences Upgrading from TypeScript 1.8 to 2.1

We're now almost 12 months into rewritting our Navy client's Apache Flex web
applications using TypeScript 1.8 with React and Redux. It's been an
exhilarating time working with modern web development technologies, practices,
and communities. However, this brave new world moves fast. In less than a year
we found ourselves needing to upgrade TypeScript just to keep our tech stack
current. Here's why and how we did it.

## Why TypeScript

A lot has already been written on why you might choose TypeScript so I won't
dive into this topic too deeply. For us the standout reasons were...

1. to enable the use of ES6 (and later) features now
1. to catch errors at compile time rather than at runtime when possible
1. to provide for improved tooling (e.g., intellisense, refactoring)

For more reasons why you might want to use TypeScript see [Why TypeScript](1)
and [Angular: Why TypeScript](2). Or search [TypeScript vs Javascript](3) for a
lot more content.

## Why TypeScript 2.1

When we looked at upgrading to TypeScript 2.1 three capabilities stood out among
the rest;

1. Simplified Declaration File Acquistion
1. Object Spread & Rest Support
1. Better Type Checking

### Simpler Declaration File Acquisition

[Simpler declaration file acquistion](4) was introduced in TypeScript 2.0. Since
2.0 all that is needed to obtain type declarations is npm.

```powershell
npm install --save @types/react
```

This will save type definition file for react to the `nodemodules/@types/react`
directory and save the dependency in package.json.

This capabilitiy eliminates the need for `tsd`, `typings`, and the `typings.json`
file. Type declaration files can be easily installed, uninstalled, and searched
with npm via standard npm install, uninstall, and view commands. You need only 
prefix the package name with the `@types/` scope!

### Object Spread & Rest Support

Since we're working with React and Redux we make heavy use of props, actions,
and reducers. Before TypeScript 2.1 to obtain a shallow copy of an object we
needed to use Object.assign.

```typescript
case actions.GENERATE_REPORT: {
    return Object.assign({}, state, {fetchStatus: "FETCHING"});
}
```

When you have a lot of actions or nontrivial objects to combine this can become
a little verbose. With `object spread` we can write our reducer like this.

```typescript
case actions.GENERATE_REPORT: {
    return {...state, fetchStatus: "FETCHING"};
}
```

The flip side of object spread is `object rest`. With `object rest` we can pluck
what we need from an object and capture what we do not need in another object.

```typescript
public render () {
    const {map, ...otherProps} = this.props;
    return (
        <div style={this.appStyle}>
            <DashboardContainer map={map}/>
            <ModalsContainer {...otherProps}/>
        </div>
    );
};
```

### Better Type Checking

We found this one out during our initial upgrade attempts due to several
compilation errors that occurred after the initial upgrade (we'll talk more
about that below).

After fixing for compilation errors the end result was a code base that was more
descriptive and self-documenting. For example, compare this code.

```typescript
// before TypeScript 2.1
export interface IDashboardItemProps {
    onExpand: Function;
}

// after TypeScript 2.1
export interface IDashboardItemProps {
    onExpand: React.EventHandler<React.MouseEvent>;
}
```

We probably could have written the more expressive version in Typescript 1.8
however the compiler allowed us to be sloppy. It never indicated that a more
narrow type was possible, or preferred.

The enhanced type checking is able to find more errors and provide more hints to
the toolchain for an improved development experience (i.e. intellisense).

## The Upgrade Process

Okay, enough of why we chose to upgrade. Let's dive into the how. For each
application or component we upgraded there were hree distinct parts; Upgrade
TypeScript, Fix Compilation Errors, and Migrate Away from Typings.

### Upgrading TypeScript and Friends

This was a straight forward process.

1. In package.json upgrade `ts-loader`, `tslint`, `typescript`, and `typings` to
   the latest available stable versions.

   ```json
   "ts-loader": "~1.3.3",
   "tslint": "~4.3.1",
   "typescript": "2.1.5",
   "typings": "~2.1.0",
   ```

1. Perform a fresh `npm install` and `typings install`

   ```powershell
   rm node-modules -force -recurse
   rm typings -force -recurse
   npm install
   typings install
   ```

   *Note: Our package.json file defines `typings install` as a `postinstall`
   step so a separate `typings install` is not strictly necessary in our setup.
   But I include it here for clarity.*

1. Build the app

   ```powershell
   npm run build
   ```

1. Admire your ~~finished work~~ compilation errors

   At this point the application would not compile cleanly. There were a variety
   of compilation errors that arose due to TypeScript 2.1's improved code flow
   analysis. Because this was the trickiest part to work through it deserves its
   own section.

### Fixing Compilation Errors

Due soley to TypeScript's improved code analysis we found ourselves with a few
dozen compilation errors across two applications and several custom components.
Fortunately, all of the errors took on a form similar to:

> Type "something" is not assignable to type "something else"

Here are a few of the more common cases we ran across.

#### Type 'string' is not assignable to type 'some | union | type'

This was the most common error caused by how we originally defined our styles.

```jsx
const container = {
    display: "flex",
    flexDirection: "column",
    justifyContent: "center"
};

return (
    <div style={container}>
        <span>Loading...</span>
    </div>
);
```

In this case `justifyContent` was inferred to be a `string` but TypeScript 2.1
knew that `style={container}>` expected `justifyContent` to be a union type.
The solution was to explictly type `container` like so.

```typescript
const container: React.CSSProperties = {
    display: "flex",
    flexDirection: "column",
    justifyContent: "center"
};
```

#### Argument of type 'Function' is not assignable to parameter of type 'EventHandler\<MouseEvent\<any\>\>'

Another common error we encounterd was defining a function too broadly. Take for
example this code.

```typescript
export interface IDashboardItemProps {
    onExpand: Function;
}

const createMenuButton = (onClick: React.EventHandler<React.MouseEvent<any>>) => {
    return <div onClick={onClick}>
            <i style={this.style} className={this.className}/>
        </div>;
};

const expandButton = (): React.ReactElement<any> => {
        return createMenuButton(this.props.onExpand);
    }
};
```

This was legitmate code in TypeScript 1.8. In TypeScript 2.1 however the
compiler was able to determine that the div's `onClick` event a
`React.EventHandler<React.MouseEvent<HTMLDivElement>>` The stricter type
analysis wouldn't have any of that. This was easily corrected with the following
tweak.

```typescript
export interface IDashboardItemProps {
    onExpand: React.EventHandler<React.MouseEvent>;
}
```

#### Type '"10"' is not assignable to type number

In the following case we incporrectly used a string to represent a number.

```jsx
return (
    <div><select size="10"></div>
);
```

TypeScript knew better and called us out on it. This fix was embarrassingly
simple.

```jsx
return (
    <div><select size=10></div>
);
```

These three types of errors made up the vast majority of the issues we
encountered upgrading from TypeScript 1.8 to TypeScript 2.1. Seeing several
dozens of compilation errors displayed was initially disheartening but
resolution for the vast majority of errors was simply to be more explict about
our types.

### Migrating Away from Typings

  - port typings from typings.json to package.json
  - finding packages
    - built in (immutable)/ how to find
    - npm view or http://microsoft.github.io/TypeSearch/ (found during the making of this blog)
  -

[1]: https://basarat.gitbooks.io/typescript/content/docs/why-typescript.html
[2]: https://vsavkin.com/writing-angular-2-in-typescript-1fa77c78d8e8#.yrgetx4n1
[3]: http://lmgtfy.com/?q=typescript+vs+javascript
[4]: https://github.com/Microsoft/TypeScript/issues/9184
[5]: https://github.com/Microsoft/TypeScript/wiki/What's-new-in-TypeScript#control-flow-based-type-analysis