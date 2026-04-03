# Purchase Order (PO) JSON Schema

This schema is used for internal procurement documents (Purchase Orders).

| Key             | Type   | Description                                                                     | If Absent |
| :-------------- | :----- | :------------------------------------------------------------------------------ | :-------- |
| `buyer`         | string | Name of the entity purchasing the goods/services.                               | `null`    |
| `supplier`      | string | Name of the entity providing the goods/services.                                | `null`    |
| `po_number`     | string | The unique PO reference number.                                                 | `null`    |
| `date`          | string | Date the PO was generated (`YYYY-MM-DD`).                                       | `null`    |
| `delivery_date` | string | Expected delivery date (`YYYY-MM-DD`).                                          | `null`    |
| `currency`      | string | 3-letter ISO currency code.                                                     | `null`    |
| `total`         | float  | Total authorized amount.                                                        | `0.0`     |
| `items`         | array  | List of objects with `item_name` (str), `quantity` (int), `unit_price` (float). | `[]`      |
