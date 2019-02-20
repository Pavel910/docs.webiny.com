---
id: admin-app-form
title: Admin App - Form
sidebar_label: Form
---

Form component in Webiny comes as a standalone package - `webiny-form`.
It is very flexible and allows to break complex forms into multiple
components and even files, thanks to its internal `Bind` component. It
also is pretty flexible when it comes to validation, both sync and async.

## Form component

Here is the `Props` type of the `Form` component:
```
type Props = {
    invalidFields?: Object,
    data?: Object,
    disabled?: boolean | Function,
    validateOnFirstSubmit?: boolean,
    submitOnEnter?: boolean,
    onSubmit?: (data: Object, form: Form) => void,
    onInvalid?: () => void,
    onChange?: (data: Object, form: Form) => void,
    children: ({
        data: Object,
        form: Form,
        submit: ({ event?: SyntheticEvent<*> }) => Promise<void>,
        Bind: React.ComponentType<BindProps>
    }) => React.Node
};
```

The simplest example looks like this:
```
<Form data={{}} onSubmit={onSubmit}>
    {({ data, form, Bind }) => (
        <React.Fragment>
            <Bind name="name" validators={["required"]}>
                <Input label="Name" />
            </Bind>
            <ButtonPrimary onClick={form.submit}>
                Submit
            </ButtonPrimary>
        </React.Fragment>
    )}
</Form>
```

`data` prop defines the initial form data. You can use it to provide default
values for the entire form.

`onSubmit` is a callback you must provide to handle form submission.
This callback will only be executed if the form data is valid.

## Bind component
`Bind` component is provided by the Form itself and is used to bind the value
of an input to the form data.

Here are all of the props `Bind` component can handle:

```
type Props = {
    name: string,
    beforeChange?: Function,
    afterChange?: Function,
    defaultValue?: any,
    validators: Array<string>,
    validationMessages?: { [string]: string },
    children: React.Element<*> | (props: Object) => React.Element<*>,
    validate: Function
};
```

This component can be used in 2 ways:
```
// With regular child element
<Bind name="myData">
    <Input label="Country" />
</Bind>

// With a render function
<Bind name="myData">
    {(inputProps) => (
        <Input label="Country" {...inputProps} />
    )}
</Bind>
```

If used with a render function, the following props are passed to it:
```
{
    form: Form,
    disabled: boolean,
    validate: () => Promise<void>,
    validation: {
        isValid: null | boolean,
        message: null | string,
        results: any
    },
    value: any,
    onChange: (value: any) => void
}
```

This component makes it very easy to create all kinds of components
bound to the Form, you could even create a component using a simple `<span>`:

```
<Bind name="myData">
    {({ value, onChange }) => (
        <span onClick={() => onChange("new value")}>{value}</span>
    )}
</Bind>
```

## Advanced usage
Here is an example of using a few more props:
- `beforeChange` will be used to convert the value to lowercase string.
- `afterChange` will simply log the new value to console.
- we will implement a custom validator to enforce a certain string format

```
const customValidator = (value) => {
    if (value.startsWith("/") && value.endsWith("/")) {
        return true;
    }

    throw new Error("Value must begin and end with a forward slash (`/`)");
};

<Bind
    name="country"
    validators={["required", customValidator]}
    beforeChange={(value, cb) => cb(value.toLowerCase())
    afterChange={value => console.log("New value", value)}
}>
    <Input label="Country"/>
</Bind>
```

Validators can also be async. You can easily perform validation by
sending a request to your API:

```
const asyncValidator = async (value) => {
    const { data } = await client.query({
        query: isSubdomainAvailable,
        variables: { subdomain }
    });

    if (data.isSubdomainAvailable.data) {
        return true;
    }

    throw new Error("This subdomain is already taken!");
};
```