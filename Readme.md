# DB Joint Purchase

## Cambios Realizados

### 1. Configuración de Productos Relacionados

Se ha añadido la opción para que los usuarios seleccionen productos relacionados en la configuración del módulo.

#### Código Modificado: `getConfigForm()`

```php
protected function getConfigForm()
{
    $options = $this->getProductOptions();
    $fields_form = array(
        'form' => array(
            'legend' => array(
                'title' => $this->l('Settings'),
                'icon' => 'icon-cogs',
            ),
            'input' => array(
                array(
                    'type' => 'color',
                    'label' => $this->l('Color general'),
                    'desc' => $this->l('Color principal utilizado en elementos del módulo'),
                    'name' => 'DBJOINT_COLOR',
                    'class' => 'disabled',
                ),
                array(
                    'type' => 'text',
                    'label' => $this->l('Productos excluidos'),
                    'desc' => $this->l('Insertar las ids de productos excluidos separados por comas. ejem: 18,25,192'),
                    'name' => 'DBJOINT_EXCLUDE',
                    'class' => 'disabled',
                ),
                array(
                    'type' => 'select',
                    'label' => $this->l('Selecciona productos relacionados'),
                    'desc' => $this->l('Busca y selecciona hasta 3 productos relacionados.'),
                    'name' => 'DBJOINT_MANUAL_RELATED_PRODUCTS[]',
                    'class' => 'chosen',
                    'multiple' => true,
                    'options' => array(
                        'query' => $options,
                        'id' => 'id_product',
                        'name' => 'name'
                    ),
                ),
            ),
            'submit' => array(
                'title' => $this->l('Save'),
            ),
        ),
    );
    return $fields_form;
}
```

### 2. Obtener Opciones de Productos Relacionados

Se ha añadido en el método getConfigFormValues() para recuperar los valores incluyendo la lista de productos relacionados seleccionados.

#### Código Modificado: `getConfigFormValues()`

```php

protected function getConfigFormValues()
{
    return array(
        'DBJOINT_COLOR' => Configuration::get('DBJOINT_COLOR'),
        'DBJOINT_EXCLUDE' => Configuration::get('DBJOINT_EXCLUDE'),
        'DBJOINT_MANUAL_RELATED_PRODUCTS' => Configuration::get('DBJOINT_MANUAL_RELATED_PRODUCTS'),
    );
}
```

### 3. Procesamiento del Formulario

Se ha modificado el método postProcess() para manejar la lógica de guardado de los productos relacionados seleccionados. Solo se permiten hasta 3 productos relacionados.

#### Código Modificado: `postProcess()`

```php
protected function postProcess()
{
    $form_values = $this->getConfigFormValues();

    foreach (array_keys($form_values) as $key) {
        Configuration::updateValue($key, Tools::getValue($key));
        if ($key == 'DBJOINT_MANUAL_RELATED_PRODUCTS') {
            $value = Tools::getValue($key);
            if (is_array($value)) {
                $value = array_slice($value, 0, 3);
                $value = implode(',', $value);
            }
            Configuration::updateValue($key, $value);
        } else {
            Configuration::updateValue($key, Tools::getValue($key));
        }
    }
}
```

### 4. Obtener Productos Relacionados

Se ha implementado el método getProductsRelated() para recuperar los productos relacionados de la base de datos.

#### Código Modificado: `getProductsRelated()`

```php
public function getProductOptions()
{
    $sql = "SELECT id_product, name FROM " . _DB_PREFIX_ . "product_lang WHERE id_lang = " . (int) $this->context->language->id;
    $results = Db::getInstance()->executeS($sql);

    return $results;
}
```
