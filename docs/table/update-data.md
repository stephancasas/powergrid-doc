# Update Data

Some PowerGrid features like [Cell Action Buttons](table/cell-action-buttons) and [Row Action Buttons](table/row-action-buttons) allow the user to modify Table data and update the database.

You will need to configure your PowerGrid Table file (e.g. `DishTable.php`) to save data.

## Usage

First you must uncomment both, the `update()` method and the `updateMessages()` method.

```php
  public function update(array $data ): bool
  {
      //...
  }
  public function updateMessages(string $status, string $field = '_default_message'): string
  {
      //...
  }
```

The `update()` method will receive data from your `field` and try to update it in your database.

Let's take the following example:

The column "Name" reads the field `name` and is configured to [Edit on click](table/cell-action-buttons?id=editonclickbool-iseditable).

```php
Column::add()
    ->title('Dish name')
    ->field('name')
    ->editOnClick(),
```

When the user edits a dish name, the `update()` method will "catch" all data sent for the that row ID and perform a database update on `name` for this row.

!> **❗ Important:** You must treat and validate all data before the update query takes place. Additionally, you can also verify if the user has permission to edit data.

---

## Custom columns

If your Table has [Custom Columns](table/add-columns?id=closure-examples), you must modify the `$data['field']` specifying the database field where the data will be saved.

For instance, the custom column `name_uppercase` must update the database field `name`. See the example below:

```php
public function addColumns(): ?PowerGridEloquent
{
  return PowerGrid::eloquent()
    ->addColumn('name')
    ->addColumn('name_uppercase', function (Dish $model) {
      return strtoupper($model->name);
    });
}

public function update(array $data): bool
{
    //Read from column name_uppercase
    if ($data['field'] == 'name_uppercase') {
          $data['field'] = 'name'; // Update the database field name
    }
    //...
```

---

## Treating data

Some data needs to be treated and formatted to fit your database field type.

In the following example, the user sends the field `price_formatted` (`4.947,70 €`) but the database requires a decimal number (`4947.70`) to be saved in the `price` field.

A similar situation happens when editing dates: the date is sent as `dd/mm/yyyy` but the database expects `yyyy-mm-dddd`.

PowerGrid will NOT perform this conversion automatically. You must treat this data in your code, parsing and converting the value and saving on the correct database field.

```php
public function update(array $data): bool
{
    // Gets price_formatted (4.947,70 €) and convert to price (44947.70).
    
    if ($data['field'] == 'price_formatted') {
          $data['field'] = 'price'; //Update the database field price
          $data['value'] = Str::of($data['value'])
            ->replace('.', '')
            ->replace(',', '.')
            ->replaceMatches('/[^Z0-9\.]/', '');
    }

      //Parses the date from d/m.Y (25/05/2021) 

      if ($data['field'] == 'created_at_formatted' && $data['value'] != '') {
        $data['field'] = 'created_at'; // Updates the database field created_at
        $data['value'] =  Carbon::createFromFormat('d/m/Y', $data['value']);
      }
      
  try {
      // Update query
      $updated = Dish::query()
        ->find($data['id'])
        ->update([
          $data['field'] => $data['value']
        ]);
  } catch (QueryException $exception) {
      $updated = false;
  }

  return $updated;
}
```

---

## Reload data after update

To reload data after a successful update, add `$this->fillData()` inside the `update()` method.

This might be useful when the data is changed with [Edit on click](table/cell-action-buttons?=editonclickbool-iseditable) and the table must be re-sorted.

Example:

```php
public function update(array $data): bool
{
  //...

  try {
      // Update query
      $updated = Dish::query()
        ->find($data['id'])
        ->update([
          $data['field'] => $data['value']
        ]);
  } catch (QueryException $exception) {
      $updated = false;
  }

  // Reload data after a successful update
  if ($updated) {
      $this->fillData();
  }
  
  return $updated;
}
```

---

## Update Messages

By default, PowerGrid displays `success` or `error` messages after the updating process is finished.

A `_default_message` key is provided with a generic message to be used for all fields.

Custom messages can be configured for specific fields (columns) inside the `updateMessages()` method.

The following example shows the generic message and custom messages for `name` and `price` field.

```php
public function updateMessages(string $status, string $field = '_default_message'): string
{
    $updateMessages = [
        'success'   => [
          '_default_message' => __('Data has been updated successfully!'),

          //'custom_field' => __('Success updating custom field.'),
          'name' => 'Dish name updated successfully!'), // Custom message for name field
          'price' => 'Price updated! Inform the chef!'), // Custom message for price field
          
          ],

        "error" => [
          '_default_message' => __('Error updating the data.'),

          //'custom_field' => __('Error updating custom field.'),
          'price' => 'Error updating price, contact the support team!'), // Custom message for price field
        ]

    ];
    //...
```

---

## Disable Update Messages

If you wish to disable update messages, you may override `$showUpdateMessages` in your PowerGrid class.

```php
class DishesTable extends PowerGridComponent
{
    //Display Update messages
    $showUpdateMessages = false
```
<hr/>
<footer style="float: right; font-size: larger">
    <span><a style="text-decoration: none;" href="#/table/queue-export">Next →</a></span>
</footer>
